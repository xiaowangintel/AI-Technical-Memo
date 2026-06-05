# selector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/selector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AttentionSelectorConfig`, `get_attn_backend`, `_cached_get_attn_backend` for the V1 `attention` subsystem. / 为 V1 的 `attention` 子系统实现 `AttentionSelectorConfig`, `get_attn_backend`, `_cached_get_attn_backend`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from functools import cache
from typing import NamedTuple, cast, get_args

import torch

import vllm.envs as envs
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.utils.import_utils import resolve_obj_by_qualname
from vllm.v1.attention.backend import AttentionBackend, AttentionType
from vllm.v1.attention.backends.registry import (
    MambaAttentionBackendEnum,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `AttentionSelectorConfig` class / `AttentionSelectorConfig` 类
```python
class AttentionSelectorConfig(NamedTuple):
    head_size: int
    dtype: torch.dtype
    kv_cache_dtype: CacheDType | None
    block_size: int | None
    use_mla: bool = False
    has_sink: bool = False
    use_sparse: bool = False
    use_mm_prefix: bool = False
    use_per_head_quant_scales: bool = False
    attn_type: str = AttentionType.DECODER
    use_non_causal: bool = False
    use_batch_invariant: bool = False
```
**EN:** Introduces the `AttentionSelectorConfig` class on top of `NamedTuple`. Core methods include `__repr__`.
**CN:** 这里定义 `AttentionSelectorConfig` 类，其基类包括 `NamedTuple`。核心方法包括 `__repr__`。

### `AttentionSelectorConfig.__repr__` method / `AttentionSelectorConfig.__repr__` 方法
```python
    def __repr__(self):
        return (
            f"AttentionSelectorConfig(head_size={self.head_size}, "
            f"dtype={self.dtype}, "
            f"kv_cache_dtype={self.kv_cache_dtype}, "
            f"block_size={self.block_size}, "
            f"use_mla={self.use_mla}, "
            f"has_sink={self.has_sink}, "
            f"use_sparse={self.use_sparse}, "
            f"use_mm_prefix={self.use_mm_prefix}, "
            f"use_per_head_quant_scales={self.use_per_head_quant_scales}, "
            f"attn_type={self.attn_type}, "
            f"use_non_causal={self.use_non_causal}, "
            f"use_batch_invariant={self.use_batch_invariant})"
        )
```
**EN:** This method implements `__repr__` within `AttentionSelectorConfig`.
**CN:** 该方法会实现 `__repr__`，其作用域位于`AttentionSelectorConfig`。

### `get_attn_backend` function / `get_attn_backend` 函数
```python
def get_attn_backend(
    head_size: int,
    dtype: torch.dtype,
    kv_cache_dtype: str | None,
    use_mla: bool = False,
    has_sink: bool = False,
    use_sparse: bool = False,
    use_mm_prefix: bool = False,
    use_per_head_quant_scales: bool = False,
    attn_type: str | None = None,
    num_heads: int | None = None,
) -> type[AttentionBackend]:
    """Selects which attention backend to use and lazily imports it."""

    if kv_cache_dtype is not None:
        valid_cache_dtypes = get_args(CacheDType)
        assert kv_cache_dtype in valid_cache_dtypes, (
            f"Invalid kv_cache_dtype: {kv_cache_dtype}. "
            f"Valid values are: {valid_cache_dtypes}"
        )

    from vllm.config import get_current_vllm_config

    vllm_config = get_current_vllm_config()

    cache_config = vllm_config.cache_config
    if cache_config is not None and cache_config.user_specified_block_size:
        block_size = cache_config.block_size
    else:
        block_size = None

    attn_selector_config = AttentionSelectorConfig(
        head_size=head_size,
        dtype=dtype,
        kv_cache_dtype=cast(CacheDType | None, kv_cache_dtype),
        block_size=block_size,
        use_mla=use_mla,
        has_sink=has_sink,
        use_sparse=use_sparse,
        use_mm_prefix=use_mm_prefix,
        use_per_head_quant_scales=use_per_head_quant_scales,
        attn_type=attn_type or AttentionType.DECODER,
        use_non_causal=vllm_config.attention_config.use_non_causal,
        use_batch_invariant=envs.VLLM_BATCH_INVARIANT,
    )

    return _cached_get_attn_backend(
        backend=vllm_config.attention_config.backend,
        attn_selector_config=attn_selector_config,
        num_heads=num_heads,
    )
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Selects which attention backend to use and lazily imports it. Key calls include `get_current_vllm_config`, `AttentionSelectorConfig`, `_cached_get_attn_backend`, `get_args`, `cast`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_current_vllm_config`, `AttentionSelectorConfig`, `_cached_get_attn_backend`, `get_args`, `cast`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_cached_get_attn_backend` function / `_cached_get_attn_backend` 函数
```python
@cache
def _cached_get_attn_backend(
    backend,
    attn_selector_config: AttentionSelectorConfig,
    num_heads: int | None = None,
) -> type[AttentionBackend]:
    from vllm.platforms import current_platform

    attention_cls = current_platform.get_attn_backend_cls(
        backend,
        attn_selector_config=attn_selector_config,
        num_heads=num_heads,
    )
    if not attention_cls:
        raise ValueError(
            f"Invalid attention backend for {current_platform.device_name}"
        )
    backend = resolve_obj_by_qualname(attention_cls)

    # Adjust kv cache layout if the selected backend requires a specific one
    required_layout = backend.get_required_kv_cache_layout()
    if required_layout is not None:
        from vllm.v1.attention.backends.utils import set_kv_cache_layout

        set_kv_cache_layout(required_layout)
        logger.info(
            "Using %s KV cache layout for %s backend.",
            required_layout,
            backend.get_name(),
        )

    return backend
```
**EN:** This function implements `_cached_get_attn_backend` within the module. Key calls include `get_attn_backend_cls`, `resolve_obj_by_qualname`, `get_required_kv_cache_layout`, `ValueError`, `set_kv_cache_layout`, `info`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_cached_get_attn_backend`，其作用域位于the module。 关键调用包括 `get_attn_backend_cls`, `resolve_obj_by_qualname`, `get_required_kv_cache_layout`, `ValueError`, `set_kv_cache_layout`, `info`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_mamba_attn_backend` function / `get_mamba_attn_backend` 函数
```python
def get_mamba_attn_backend(
    mamba_type: MambaAttentionBackendEnum,
) -> type[AttentionBackend]:
    """Select which mamba attention backend to use and lazily import it."""
    return _cached_get_mamba_attn_backend(mamba_type)
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Select which mamba attention backend to use and lazily import it. Key calls include `_cached_get_mamba_attn_backend`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `_cached_get_mamba_attn_backend`。

### `_cached_get_mamba_attn_backend` function / `_cached_get_mamba_attn_backend` 函数
```python
@cache
def _cached_get_mamba_attn_backend(
    mamba_type: MambaAttentionBackendEnum,
) -> type[AttentionBackend]:
    assert mamba_type and isinstance(mamba_type, MambaAttentionBackendEnum)

    mamba_attn_backend = mamba_type.get_class()
    if envs.VLLM_BATCH_INVARIANT and not mamba_attn_backend.supports_batch_invariance():
        raise RuntimeError(
            "VLLM batch_invariant mode is not supported for "
            f"{mamba_attn_backend.get_name()}."
        )
    return mamba_attn_backend
```
**EN:** This function implements `_cached_get_mamba_attn_backend` within the module. Key calls include `get_class`, `isinstance`, `RuntimeError`, `supports_batch_invariance`, `get_name`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_cached_get_mamba_attn_backend`，其作用域位于the module。 关键调用包括 `get_class`, `isinstance`, `RuntimeError`, `supports_batch_invariance`, `get_name`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `AttentionSelectorConfig`: central class or interface in this module. / `AttentionSelectorConfig`：本模块中的核心类或接口。
- `get_attn_backend`: top-level helper or orchestration entry point. / `get_attn_backend`：顶层辅助函数或编排入口。
- `_cached_get_attn_backend`: top-level helper or orchestration entry point. / `_cached_get_attn_backend`：顶层辅助函数或编排入口。
- `get_mamba_attn_backend`: top-level helper or orchestration entry point. / `get_mamba_attn_backend`：顶层辅助函数或编排入口。
- `_cached_get_mamba_attn_backend`: top-level helper or orchestration entry point. / `_cached_get_mamba_attn_backend`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `functools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config.cache`, `vllm.logger`, `vllm.utils.import_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.registry`, `vllm.config`, `vllm.platforms`, `vllm.v1.attention.backends.utils`
