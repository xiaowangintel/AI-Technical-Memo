# backend_adaptor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/sparsity/backend/backend_adaptor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the backend adaptor logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的后端适配器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Imports and setup / 导入与初始化
```python
import logging
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, Any, Optional

import torch
```
**EN:** Imports `logging`, `abc`, `typing`, `torch` and other helpers used by the surrounding scope.
**CN:** 导入 `logging`, `abc`, `typing`, `torch` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 7-8: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 10-10: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 13-15: BackendAdaptor declaration / BackendAdaptor 声明
```python
class BackendAdaptor(ABC):
    """Base class for attention backend adaptors."""

```
**EN:** Base class for attention backend adaptors. Declares the `BackendAdaptor` class and connects it to `ABC`.
**CN:** 声明 `BackendAdaptor` 类，并将其关联到 `ABC`。

### Lines 16-18: __init__ implementation / __init__ 实现
```python
    def __init__(self, device: torch.device):
        self.device = device
        self._original_metadata = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `BackendAdaptor`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `BackendAdaptor`。

### Lines 20-22: save_original_metadata implementation / save_original_metadata 实现
```python
    def save_original_metadata(self, metadata: Any) -> None:
        """Save original metadata in the beginning of the forward pass."""
        pass
```
**EN:** Save original metadata in the beginning of the forward pass. Persists the current state to an external destination. It belongs to `BackendAdaptor`.
**CN:** 将当前状态持久化到外部位置。 该方法属于 `BackendAdaptor`。

### Lines 24-46: adapt_for_attn_metadata implementation / adapt_for_attn_metadata 实现
```python
    @abstractmethod
    def adapt_for_attn_metadata(
        self,
        selected_indices: torch.Tensor,
        valid_lengths: torch.Tensor,
        sparse_mask: torch.Tensor,
        current_metadata: Any,
        forward_batch: "ForwardBatch",
        req_to_token: torch.Tensor,
        page_size: int,
        layer_id: int,
        **kwargs,
# ... omitted for brevity ...
        Returns:
            Modified attention metadata compatible with the backend
        """
        pass
```
**EN:** Adapt attention metadata for sparse KVCache access. Implements the adapt for attn metadata routine for this scope. It belongs to `BackendAdaptor`.
**CN:** 实现当前作用域中的adapt for attn元数据例程。 该方法属于 `BackendAdaptor`。

### Lines 49-51: NSABackendAdaptor declaration / NSABackendAdaptor 声明
```python
class NSABackendAdaptor(BackendAdaptor):
    """Adaptor for NSA (Native Sparse Attention) backend."""

```
**EN:** Adaptor for NSA (Native Sparse Attention) backend. Declares the `NSABackendAdaptor` class and connects it to `BackendAdaptor`.
**CN:** 声明 `NSABackendAdaptor` 类，并将其关联到 `BackendAdaptor`。

### Lines 52-58: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        device: torch.device,
        req_to_token_pool,
    ):
        super().__init__(device)
        self.req_to_token_pool = req_to_token_pool
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `NSABackendAdaptor`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `NSABackendAdaptor`。

### Lines 60-76: adapt_for_attn_metadata implementation / adapt_for_attn_metadata 实现
```python
    def adapt_for_attn_metadata(
        self,
        selected_indices: torch.Tensor,
        valid_lengths: torch.Tensor,
        sparse_mask: torch.Tensor,
        current_metadata: Any,
        forward_batch: "ForwardBatch",
        req_to_token: torch.Tensor,
        page_size: int,
        layer_id: int,
        **kwargs,
    ) -> Optional[torch.Tensor]:
        """
        Transform logical page indices to physical device indices for NSA backend.
        """
        # TODO: Implement NSA backend adaptor logic
        pass
```
**EN:** Transform logical page indices to physical device indices for NSA backend. Implements the adapt for attn metadata routine for this scope. It belongs to `NSABackendAdaptor`.
**CN:** 实现当前作用域中的adapt for attn元数据例程。 该方法属于 `NSABackendAdaptor`。

### Lines 79-81: FlashAttentionAdaptor declaration / FlashAttentionAdaptor 声明
```python
class FlashAttentionAdaptor(BackendAdaptor):
    """Adaptor for FlashAttention backend."""

```
**EN:** Adaptor for FlashAttention backend. Declares the `FlashAttentionAdaptor` class and connects it to `BackendAdaptor`.
**CN:** 声明 `FlashAttentionAdaptor` 类，并将其关联到 `BackendAdaptor`。

### Lines 82-88: save_original_metadata implementation / save_original_metadata 实现
```python
    def save_original_metadata(self, metadata: Any) -> None:
        self._original_metadata = {
            "page_table": metadata.page_table.clone(),
            "cache_seqlens_int32": metadata.cache_seqlens_int32.clone(),
            "cu_seqlens_k": metadata.cu_seqlens_k.clone(),
            "max_seq_len_k": metadata.max_seq_len_k,
        }
```
**EN:** Persists the current state to an external destination. It belongs to `FlashAttentionAdaptor`.
**CN:** 将当前状态持久化到外部位置。 该方法属于 `FlashAttentionAdaptor`。

### Lines 90-154: adapt_for_attn_metadata implementation / adapt_for_attn_metadata 实现
```python
    def adapt_for_attn_metadata(
        self,
        selected_indices: torch.Tensor,
        valid_lengths: torch.Tensor,
        sparse_mask: torch.Tensor,
        current_metadata: Any,
        forward_batch: "ForwardBatch",
        req_to_token: torch.Tensor,
        page_size: int,
        layer_id: int,
        **kwargs,
    ) -> Any:
# ... omitted for brevity ...
            (1, 0),
        )
        current_metadata.max_seq_len_k = int(current_metadata.cache_seqlens_int32.max())
        return current_metadata
```
**EN:** Adapt FlashAttention metadata for sparse KVCache access. Implements the adapt for attn metadata routine for this scope. It belongs to `FlashAttentionAdaptor`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的adapt for attn元数据例程。 该方法属于 `FlashAttentionAdaptor`。它会向调用方返回计算结果。

### Lines 156-176: _logical_to_physical_pages_batch implementation / _logical_to_physical_pages_batch 实现
```python
    def _logical_to_physical_pages_batch(
        self,
        logical_pages: torch.Tensor,
        req_pool_indices: torch.Tensor,
        req_to_token: torch.Tensor,
        page_size: int,
    ) -> torch.Tensor:
        bs, max_pages = logical_pages.shape

        page_starts = logical_pages * page_size
        page_starts_clamped = page_starts.clamp(min=0)

# ... omitted for brevity ...
            logical_pages >= 0, physical_pages, torch.zeros_like(physical_pages)
        )

        return physical_pages.to(torch.int32)
```
**EN:** Implements the logical TO physical pages batch routine for this scope. It belongs to `FlashAttentionAdaptor`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的logical to physical pages batch例程。 该方法属于 `FlashAttentionAdaptor`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`BackendAdaptor`**: Defines the `BackendAdaptor` type and its core responsibilities. / 定义 `BackendAdaptor` 类型及其核心职责。
- **`NSABackendAdaptor`**: Defines the `NSABackendAdaptor` type and its core responsibilities. / 定义 `NSABackendAdaptor` 类型及其核心职责。
- **`FlashAttentionAdaptor`**: Defines the `FlashAttentionAdaptor` type and its core responsibilities. / 定义 `FlashAttentionAdaptor` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `abc`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.model_executor.forward_batch_info`
