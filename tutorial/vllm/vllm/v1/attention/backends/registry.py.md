# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention backend registry / 该模块位于 `attention/backends` 子系统，主要围绕 `_AttentionBackendEnumMeta`, `AttentionBackendEnum`, `MambaAttentionBackendEnum` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention backend registry"""

from collections.abc import Callable
from enum import Enum, EnumMeta
from typing import TYPE_CHECKING, cast

from vllm.logger import init_logger
from vllm.utils.import_utils import resolve_obj_by_qualname

if TYPE_CHECKING:
    from vllm.v1.attention.backend import AttentionBackend

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_AttentionBackendEnumMeta` class / `_AttentionBackendEnumMeta` 类
```python
class _AttentionBackendEnumMeta(EnumMeta):
    """Metaclass for AttentionBackendEnum to provide better error messages."""
```
**EN:** Defines the `_AttentionBackendEnumMeta` enum used to normalize modes or options across the subsystem. Representative members: see class body.
**CN:** `_AttentionBackendEnumMeta` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：见类体。

### `_AttentionBackendEnumMeta.__getitem__` method / `_AttentionBackendEnumMeta.__getitem__` 方法
```python
    def __getitem__(cls, name: str):
        """Get backend by name with helpful error messages."""
        try:
            return super().__getitem__(name)
        except KeyError:
            members = cast("dict[str, Enum]", cls.__members__).keys()
            valid_backends = ", ".join(members)
            raise ValueError(
                f"Unknown attention backend: '{name}'. "
                f"Valid options are: {valid_backends}"
            ) from None
```
**EN:** This method implements `__getitem__` within `_AttentionBackendEnumMeta`. The docstring frames it as: Get backend by name with helpful error messages. Key calls include `__getitem__`, `keys`, `join`, `ValueError`, `super`, `cast`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__getitem__`，其作用域位于`_AttentionBackendEnumMeta`。 关键调用包括 `__getitem__`, `keys`, `join`, `ValueError`, `super`, `cast`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionBackendEnum` class / `AttentionBackendEnum` 类
```python
class AttentionBackendEnum(Enum, metaclass=_AttentionBackendEnumMeta):
    """Enumeration of all supported attention backends.

    The enum value is the default class path, but this can be overridden
    at runtime using register_backend().

    To get the actual backend class (respecting overrides), use:
        backend.get_class()
    """

    FLASH_ATTN = "vllm.v1.attention.backends.flash_attn.FlashAttentionBackend"
    FLASH_ATTN_DIFFKV = (
        "vllm.v1.attention.backends.flash_attn_diffkv.FlashAttentionDiffKVBackend"
    )
    TRITON_ATTN = "vllm.v1.attention.backends.triton_attn.TritonAttentionBackend"
    ROCM_ATTN = "vllm.v1.attention.backends.rocm_attn.RocmAttentionBackend"
    ROCM_AITER_MLA = "vllm.v1.attention.backends.mla.rocm_aiter_mla.AiterMLABackend"
    ROCM_AITER_TRITON_MLA = (
        "vllm.v1.attention.backends.mla.aiter_triton_mla.AiterTritonMLABackend"
    )
    ROCM_AITER_FA = (
        "vllm.v1.attention.backends.rocm_aiter_fa.AiterFlashAttentionBackend"
    )
    ROCM_AITER_MLA_SPARSE = (
        "vllm.v1.attention.backends.mla.rocm_aiter_mla_sparse.ROCMAiterMLASparseBackend"
    )
    XPU_MLA_SPARSE = "vllm.v1.attention.backends.mla.xpu_mla_sparse.XPUMLASparseBackend"
    TORCH_SDPA = ""  # this tag is only used for ViT
    FLASHINFER = "vllm.v1.attention.backends.flashinfer.FlashInferBackend"
    FLASHINFER_MLA = (
        "vllm.v1.attention.backends.mla.flashinfer_mla.FlashInferMLABackend"
    )
    TOKENSPEED_MLA = (
        "vllm.v1.attention.backends.mla.tokenspeed_mla.TokenspeedMLABackend"
    )
    FLASHINFER_MLA_SPARSE = (
        "vllm.v1.attention.backends.mla.flashinfer_mla_sparse."
        "FlashInferMLASparseBackend"
    )
    TRITON_MLA = "vllm.v1.attention.backends.mla.triton_mla.TritonMLABackend"
    CUTLASS_MLA = "vllm.v1.attention.backends.mla.cutlass_mla.CutlassMLABackend"
    FLASHMLA = "vllm.v1.attention.backends.mla.flashmla.FlashMLABackend"
    FLASHMLA_SPARSE = (
        "vllm.v1.attention.backends.mla.flashmla_sparse.FlashMLASparseBackend"
    )
    FLASH_ATTN_MLA = "vllm.v1.attention.backends.mla.flashattn_mla.FlashAttnMLABackend"
    NO_ATTENTION = "vllm.v1.attention.backends.no_attention.NoAttentionBackend"
    FLEX_ATTENTION = "vllm.v1.attention.backends.flex_attention.FlexAttentionBackend"
    ROCM_AITER_UNIFIED_ATTN = (
        "vllm.v1.attention.backends.rocm_aiter_unified_attn."
        "RocmAiterUnifiedAttentionBackend"
    )
    CPU_ATTN = "vllm.v1.attention.backends.cpu_attn.CPUAttentionBackend"
    TURBOQUANT = "vllm.v1.attention.backends.turboquant_attn.TurboQuantAttentionBackend"
    # Placeholder for third-party/custom backends - must be registered before use
    # set to None to avoid alias with other backend, whose value is an empty string
    CUSTOM = None
```
**EN:** Defines the `AttentionBackendEnum` enum used to normalize modes or options across the subsystem. Representative members: `FLASH_ATTN`, `FLASH_ATTN_DIFFKV`, `TRITON_ATTN`, `ROCM_ATTN`, `ROCM_AITER_MLA`, `ROCM_AITER_TRITON_MLA`.
**CN:** `AttentionBackendEnum` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`FLASH_ATTN`, `FLASH_ATTN_DIFFKV`, `TRITON_ATTN`, `ROCM_ATTN`, `ROCM_AITER_MLA`, `ROCM_AITER_TRITON_MLA`。

### `AttentionBackendEnum.get_path` method / `AttentionBackendEnum.get_path` 方法
```python
    def get_path(self, include_classname: bool = True) -> str:
        """Get the class path for this backend (respects overrides).

        Returns:
            The fully qualified class path string

        Raises:
            ValueError: If Backend.CUSTOM is used without being registered
        """
        path = _ATTN_OVERRIDES.get(self, self.value)
        if not path:
            raise ValueError(
                f"Backend {self.name} must be registered before use. "
                f"Use register_backend(Backend.{self.name}, 'your.module.YourClass')"
            )
        if not include_classname:
            path = path.rsplit(".", 1)[0]
        return path
```
**EN:** This method returns or derives a value within `AttentionBackendEnum`. The docstring frames it as: Get the class path for this backend (respects overrides). Key calls include `get`, `ValueError`, `rsplit`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionBackendEnum`。 关键调用包括 `get`, `ValueError`, `rsplit`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionBackendEnum.get_class` method / `AttentionBackendEnum.get_class` 方法
```python
    def get_class(self) -> "type[AttentionBackend]":
        """Get the backend class (respects overrides).

        Returns:
            The backend class

        Raises:
            ImportError: If the backend class cannot be imported
            ValueError: If Backend.CUSTOM is used without being registered
        """
        return resolve_obj_by_qualname(self.get_path())
```
**EN:** This method returns or derives a value within `AttentionBackendEnum`. The docstring frames it as: Get the backend class (respects overrides). Key calls include `resolve_obj_by_qualname`, `get_path`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionBackendEnum`。 关键调用包括 `resolve_obj_by_qualname`, `get_path`。

### `AttentionBackendEnum.is_overridden` method / `AttentionBackendEnum.is_overridden` 方法
```python
    def is_overridden(self) -> bool:
        """Check if this backend has been overridden.

        Returns:
            True if the backend has a registered override
        """
        return self in _ATTN_OVERRIDES
```
**EN:** This method answers a boolean capability check within `AttentionBackendEnum`. The docstring frames it as: Check if this backend has been overridden.
**CN:** 该方法会回答布尔能力判断，其作用域位于`AttentionBackendEnum`。

### `AttentionBackendEnum.clear_override` method / `AttentionBackendEnum.clear_override` 方法
```python
    def clear_override(self) -> None:
        """Clear any override for this backend, reverting to the default."""
        _ATTN_OVERRIDES.pop(self, None)
```
**EN:** This method implements `clear_override` within `AttentionBackendEnum`. The docstring frames it as: Clear any override for this backend, reverting to the default. Key calls include `pop`.
**CN:** 该方法会实现 `clear_override`，其作用域位于`AttentionBackendEnum`。 关键调用包括 `pop`。

### `MambaAttentionBackendEnum` class / `MambaAttentionBackendEnum` 类
```python
class MambaAttentionBackendEnum(Enum, metaclass=_AttentionBackendEnumMeta):
    """Enumeration of all supported mamba attention backends.

    The enum value is the default class path, but this can be overridden
    at runtime using register_backend().

    To get the actual backend class (respecting overrides), use:
        backend.get_class()
    """

    MAMBA1 = "vllm.v1.attention.backends.mamba1_attn.Mamba1AttentionBackend"
    MAMBA2 = "vllm.v1.attention.backends.mamba2_attn.Mamba2AttentionBackend"
    SHORT_CONV = "vllm.v1.attention.backends.short_conv_attn.ShortConvAttentionBackend"
    LINEAR = "vllm.v1.attention.backends.linear_attn.LinearAttentionBackend"
    GDN_ATTN = "vllm.v1.attention.backends.gdn_attn.GDNAttentionBackend"
    # Placeholder for third-party/custom backends - must be registered before use
    # set to None to avoid alias with other backend, whose value is an empty string
    CUSTOM = None
```
**EN:** Defines the `MambaAttentionBackendEnum` enum used to normalize modes or options across the subsystem. Representative members: `MAMBA1`, `MAMBA2`, `SHORT_CONV`, `LINEAR`, `GDN_ATTN`, `CUSTOM`.
**CN:** `MambaAttentionBackendEnum` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`MAMBA1`, `MAMBA2`, `SHORT_CONV`, `LINEAR`, `GDN_ATTN`, `CUSTOM`。

### `MambaAttentionBackendEnum.get_path` method / `MambaAttentionBackendEnum.get_path` 方法
```python
    def get_path(self, include_classname: bool = True) -> str:
        """Get the class path for this backend (respects overrides).

        Returns:
            The fully qualified class path string

        Raises:
            ValueError: If Backend.CUSTOM is used without being registered
        """
        path = _MAMBA_ATTN_OVERRIDES.get(self, self.value)
        if not path:
            raise ValueError(
                f"Backend {self.name} must be registered before use. "
                f"Use register_backend(Backend.{self.name}, 'your.module.YourClass')"
            )
        if not include_classname:
            path = path.rsplit(".", 1)[0]
        return path
```
**EN:** This method returns or derives a value within `MambaAttentionBackendEnum`. The docstring frames it as: Get the class path for this backend (respects overrides). Key calls include `get`, `ValueError`, `rsplit`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`MambaAttentionBackendEnum`。 关键调用包括 `get`, `ValueError`, `rsplit`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MambaAttentionBackendEnum.get_class` method / `MambaAttentionBackendEnum.get_class` 方法
```python
    def get_class(self) -> "type[AttentionBackend]":
        """Get the backend class (respects overrides).

        Returns:
            The backend class

        Raises:
            ImportError: If the backend class cannot be imported
            ValueError: If Backend.CUSTOM is used without being registered
        """
        return resolve_obj_by_qualname(self.get_path())
```
**EN:** This method returns or derives a value within `MambaAttentionBackendEnum`. The docstring frames it as: Get the backend class (respects overrides). Key calls include `resolve_obj_by_qualname`, `get_path`.
**CN:** 该方法会返回或推导一个值，其作用域位于`MambaAttentionBackendEnum`。 关键调用包括 `resolve_obj_by_qualname`, `get_path`。

### `MambaAttentionBackendEnum.is_overridden` method / `MambaAttentionBackendEnum.is_overridden` 方法
```python
    def is_overridden(self) -> bool:
        """Check if this backend has been overridden.

        Returns:
            True if the backend has a registered override
        """
        return self in _MAMBA_ATTN_OVERRIDES
```
**EN:** This method answers a boolean capability check within `MambaAttentionBackendEnum`. The docstring frames it as: Check if this backend has been overridden.
**CN:** 该方法会回答布尔能力判断，其作用域位于`MambaAttentionBackendEnum`。

### `MambaAttentionBackendEnum.clear_override` method / `MambaAttentionBackendEnum.clear_override` 方法
```python
    def clear_override(self) -> None:
        """Clear any override for this backend, reverting to the default."""
        _MAMBA_ATTN_OVERRIDES.pop(self, None)
```
**EN:** This method implements `clear_override` within `MambaAttentionBackendEnum`. The docstring frames it as: Clear any override for this backend, reverting to the default. Key calls include `pop`.
**CN:** 该方法会实现 `clear_override`，其作用域位于`MambaAttentionBackendEnum`。 关键调用包括 `pop`。

### Module constants / 模块常量
```python
_ATTN_OVERRIDES: dict[AttentionBackendEnum, str] = {}
_MAMBA_ATTN_OVERRIDES: dict[MambaAttentionBackendEnum, str] = {}
```
**EN:** Defines module-level constants or aliases such as `_ATTN_OVERRIDES`, `_MAMBA_ATTN_OVERRIDES`, which are reused by later definitions.
**CN:** 定义 `_ATTN_OVERRIDES`, `_MAMBA_ATTN_OVERRIDES` 等模块级常量或别名，供后续定义复用。

### `register_backend` function / `register_backend` 函数
```python
def register_backend(
    backend: AttentionBackendEnum | MambaAttentionBackendEnum,
    class_path: str | None = None,
    is_mamba: bool = False,
) -> Callable[[type], type]:
    """Register or override a backend implementation.

    Args:
        backend: The AttentionBackendEnum member to register
        class_path: Optional class path. If not provided and used as
            decorator, will be auto-generated from the class.

    Returns:
        Decorator function if class_path is None, otherwise a no-op

    Examples:
        # Override an existing attention backend
        @register_backend(AttentionBackendEnum.FLASH_ATTN)
        class MyCustomFlashAttn:
            ...

        # Override an existing mamba attention backend
        @register_backend(MambaAttentionBackendEnum.LINEAR, is_mamba=True)
        class MyCustomMambaAttn:
            ...

        # Register a custom third-party attention backend
        @register_backend(AttentionBackendEnum.CUSTOM)
        class MyCustomBackend:
            ...

        # Direct registration
        register_backend(
            AttentionBackendEnum.CUSTOM,
            "my.module.MyCustomBackend"
        )
    """

    def decorator(cls: type) -> type:
        if is_mamba:
            _MAMBA_ATTN_OVERRIDES[backend] = f"{cls.__module__}.{cls.__qualname__}"  # type: ignore[index]
        else:
            _ATTN_OVERRIDES[backend] = f"{cls.__module__}.{cls.__qualname__}"  # type: ignore[index]
        return cls

    if class_path is not None:
        if is_mamba:
            _MAMBA_ATTN_OVERRIDES[backend] = class_path  # type: ignore[index]
        else:
            _ATTN_OVERRIDES[backend] = class_path  # type: ignore[index]
        return lambda x: x

    return decorator
```
**EN:** This function implements `register_backend` within the module. The docstring frames it as: Register or override a backend implementation. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `register_backend`，其作用域位于the module。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_AttentionBackendEnumMeta`: central class or interface in this module. / `_AttentionBackendEnumMeta`：本模块中的核心类或接口。
- `AttentionBackendEnum`: central class or interface in this module. / `AttentionBackendEnum`：本模块中的核心类或接口。
- `MambaAttentionBackendEnum`: central class or interface in this module. / `MambaAttentionBackendEnum`：本模块中的核心类或接口。
- `register_backend`: top-level helper or orchestration entry point. / `register_backend`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `enum`, `typing`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.utils.import_utils`, `vllm.v1.attention.backend`
