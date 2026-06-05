# flashmla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/flashmla.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_is_flashmla_available`, `is_flashmla_dense_supported`, `is_flashmla_sparse_supported` for the V1 `attention/ops` subsystem. / 为 V1 的 `attention/ops` 子系统实现 `_is_flashmla_available`, `is_flashmla_dense_supported`, `is_flashmla_sparse_supported`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

from vllm.logger import init_logger
from vllm.platforms import current_platform

logger = init_logger(__name__)

if current_platform.is_cuda():
    try:
        import vllm._flashmla_C  # noqa: F401

        _flashmla_C_AVAILABLE = True
    except ImportError:
        _flashmla_C_AVAILABLE = False
else:
    _flashmla_C_AVAILABLE = False

if current_platform.is_cuda():
    try:
        import vllm._flashmla_extension_C  # noqa: F401

        _flashmla_extension_C_AVAILABLE = True
    except ImportError:
        _flashmla_extension_C_AVAILABLE = False
else:
    _flashmla_extension_C_AVAILABLE = False
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_is_flashmla_available` function / `_is_flashmla_available` 函数
```python
def _is_flashmla_available() -> tuple[bool, str | None]:
    if not _flashmla_C_AVAILABLE:
        return (
            False,
            "vllm._flashmla_C is not available, likely was not "
            "compiled due to insufficient nvcc version or a supported arch "
            "was not in the list of target arches to compile for.",
        )
    if not _flashmla_extension_C_AVAILABLE:
        return (
            False,
            "vllm._flashmla_extension_C is not available, likely "
            "was not compiled due to a build error.",
        )

    return True, None
```
**EN:** This function implements `_is_flashmla_available` within the module. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_is_flashmla_available`，其作用域位于the module。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `is_flashmla_dense_supported` function / `is_flashmla_dense_supported` 函数
```python
def is_flashmla_dense_supported() -> tuple[bool, str | None]:
    """
    Return: is_supported_flag, unsupported_reason (optional).
    """
    is_available, maybe_reason = _is_flashmla_available()
    if not is_available:
        return False, maybe_reason
    if not current_platform.is_device_capability_family(90):
        return False, "FlashMLA Dense is only supported on Hopper devices."
    return True, None
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Return: is_supported_flag, unsupported_reason (optional). Key calls include `_is_flashmla_available`, `is_device_capability_family`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `_is_flashmla_available`, `is_device_capability_family`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `is_flashmla_sparse_supported` function / `is_flashmla_sparse_supported` 函数
```python
def is_flashmla_sparse_supported() -> tuple[bool, str | None]:
    """
    Return: is_supported_flag, unsupported_reason (optional).
    """
    is_available, maybe_reason = _is_flashmla_available()
    if not is_available:
        return False, maybe_reason
    if not (
        current_platform.is_device_capability_family(90)
        or current_platform.is_device_capability_family(100)
    ):
        return (
            False,
            "FlashMLA Sparse is only supported on Hopper and Blackwell devices.",
        )
    return True, None
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Return: is_supported_flag, unsupported_reason (optional). Key calls include `_is_flashmla_available`, `is_device_capability_family`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 关键调用包括 `_is_flashmla_available`, `is_device_capability_family`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_raise_flashmla_unavailable` function / `_raise_flashmla_unavailable` 函数
```python
def _raise_flashmla_unavailable(*_args, **_kwargs):
    _, reason = _is_flashmla_available()
    raise RuntimeError(reason or "FlashMLA is not available")
```
**EN:** This function implements `_raise_flashmla_unavailable` within the module. Key calls include `_is_flashmla_available`, `RuntimeError`.
**CN:** 该函数会实现 `_raise_flashmla_unavailable`，其作用域位于the module。 关键调用包括 `_is_flashmla_available`, `RuntimeError`。

### Imports and typing / 导入与类型定义
```python
if _is_flashmla_available()[0]:
    from vllm.third_party.flashmla.flash_mla_interface import (  # noqa: F401
        FlashMLASchedMeta,
        flash_attn_varlen_func,
        flash_attn_varlen_kvpacked_func,
        flash_attn_varlen_qkvpacked_func,
        flash_mla_sparse_fwd,
        flash_mla_with_kvcache,
        get_mla_metadata,
    )
else:

    class FlashMLASchedMeta:  # type: ignore[no-redef]
        pass

    flash_attn_varlen_func = _raise_flashmla_unavailable  # type: ignore[assignment]
    flash_attn_varlen_kvpacked_func = _raise_flashmla_unavailable  # type: ignore[assignment]
    flash_attn_varlen_qkvpacked_func = _raise_flashmla_unavailable  # type: ignore[assignment]
    flash_mla_sparse_fwd = _raise_flashmla_unavailable  # type: ignore[assignment]
    flash_mla_with_kvcache = _raise_flashmla_unavailable  # type: ignore[assignment]
    get_mla_metadata = _raise_flashmla_unavailable  # type: ignore[assignment]
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.logger`, `vllm.platforms`, `vllm._flashmla_C`, `vllm._flashmla_extension_C`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.logger`, `vllm.platforms`, `vllm._flashmla_C`, `vllm._flashmla_extension_C` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `get_mla_metadata_dense_fp8` function / `get_mla_metadata_dense_fp8` 函数
```python
def get_mla_metadata_dense_fp8(
    cache_seqlens: torch.Tensor,
    num_q_tokens_per_head_k: int,
    num_heads_k: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    if not _is_flashmla_available()[0]:
        _raise_flashmla_unavailable()
    return torch.ops._flashmla_extension_C.get_mla_decoding_metadata_dense_fp8(
        cache_seqlens,
        num_q_tokens_per_head_k,
        num_heads_k,
    )
```
**EN:** This function returns or derives a value within the module. Key calls include `get_mla_decoding_metadata_dense_fp8`, `_raise_flashmla_unavailable`, `_is_flashmla_available`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_mla_decoding_metadata_dense_fp8`, `_raise_flashmla_unavailable`, `_is_flashmla_available`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `flash_mla_with_kvcache_fp8` function / `flash_mla_with_kvcache_fp8` 函数
```python
def flash_mla_with_kvcache_fp8(
    q: torch.Tensor,
    k_cache: torch.Tensor,
    block_table: torch.Tensor,
    cache_seqlens: torch.Tensor,
    head_dim_v: int,
    tile_scheduler_metadata: torch.Tensor,
    num_splits: torch.Tensor,
    softmax_scale: float | None = None,
    causal: bool = False,
    descale_q: torch.Tensor | None = None,
    descale_k: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    if not _is_flashmla_available()[0]:
        _raise_flashmla_unavailable()
    if softmax_scale is None:
        softmax_scale = q.shape[-1] ** (-0.5)
    out, softmax_lse = torch.ops._flashmla_extension_C.fwd_kvcache_mla_fp8(
        q,
        k_cache,
        head_dim_v,
        cache_seqlens,
        block_table,
        softmax_scale,
        causal,
        tile_scheduler_metadata,
        num_splits,
        descale_q,
        descale_k,
    )
    return out, softmax_lse
```
**EN:** This function implements `flash_mla_with_kvcache_fp8` within the module. Key calls include `fwd_kvcache_mla_fp8`, `_raise_flashmla_unavailable`, `_is_flashmla_available`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flash_mla_with_kvcache_fp8`，其作用域位于the module。 关键调用包括 `fwd_kvcache_mla_fp8`, `_raise_flashmla_unavailable`, `_is_flashmla_available`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_is_flashmla_available`: top-level helper or orchestration entry point. / `_is_flashmla_available`：顶层辅助函数或编排入口。
- `is_flashmla_dense_supported`: top-level helper or orchestration entry point. / `is_flashmla_dense_supported`：顶层辅助函数或编排入口。
- `is_flashmla_sparse_supported`: top-level helper or orchestration entry point. / `is_flashmla_sparse_supported`：顶层辅助函数或编排入口。
- `_raise_flashmla_unavailable`: top-level helper or orchestration entry point. / `_raise_flashmla_unavailable`：顶层辅助函数或编排入口。
- `get_mla_metadata_dense_fp8`: top-level helper or orchestration entry point. / `get_mla_metadata_dense_fp8`：顶层辅助函数或编排入口。
- `flash_mla_with_kvcache_fp8`: top-level helper or orchestration entry point. / `flash_mla_with_kvcache_fp8`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.platforms`, `vllm._flashmla_C`, `vllm._flashmla_extension_C`, `vllm.third_party.flashmla.flash_mla_interface`
