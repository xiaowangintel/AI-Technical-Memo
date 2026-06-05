# sparse_coordinator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/sparsity/core/sparse_coordinator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the sparse coordinator logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的稀疏协调器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and setup / 导入与初始化
```python
import logging
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any, Optional

import torch

from sglang.srt.mem_cache.memory_pool import KVCache, ReqToTokenPool
from sglang.srt.mem_cache.sparsity.algorithms.base_algorithm import BaseSparseAlgorithm
from sglang.srt.mem_cache.sparsity.backend.backend_adaptor import BackendAdaptor
```
**EN:** Imports `logging`, `dataclasses`, `typing`, `torch`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm` and other helpers used by the surrounding scope.
**CN:** 导入 `logging`, `dataclasses`, `typing`, `torch`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 11-14: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.managers.schedule_batch import Req
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 16-16: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 19-21: RequestTrackers declaration / RequestTrackers 声明
```python
class RequestTrackers:
    """State tracker for sparse attention requests."""

```
**EN:** State tracker for sparse attention requests. Declares the `RequestTrackers` class.
**CN:** 声明 `RequestTrackers` 类。

### Lines 22-39: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        max_pool_size: int,
        device: torch.device,
        num_layers: int,
        min_sparse_prompt_len: int,
        max_context_len: int,
    ):
        self.device = device
        self.num_layers = num_layers

        self.repr_constructed = torch.zeros(
            max_pool_size, dtype=torch.bool, device=device
        )
        self.prompt_lens = torch.zeros(max_pool_size, dtype=torch.int64, device=device)
        self.last_constructed_page = torch.zeros(
            max_pool_size, dtype=torch.int64, device=device
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `RequestTrackers`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `RequestTrackers`。

### Lines 40-42: Comment block / 注释块
```python

        # TODO: Add more trackers for hierarchical KVCache management

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 43-46: register implementation / register 实现
```python
    def register(self, idx: int, prompt_len: int) -> None:
        self.repr_constructed[idx] = False
        self.prompt_lens[idx] = prompt_len
        self.last_constructed_page[idx] = 0
```
**EN:** Registers metadata so other components can discover this object. It belongs to `RequestTrackers`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `RequestTrackers`。

### Lines 48-51: clear implementation / clear 实现
```python
    def clear(self, idx: int) -> None:
        self.repr_constructed[idx] = False
        self.prompt_lens[idx] = 0
        self.last_constructed_page[idx] = 0
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `RequestTrackers`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `RequestTrackers`。

### Lines 54-57: SparseConfig declaration / SparseConfig 声明
```python
@dataclass
class SparseConfig:
    """Configuration for sparse attention."""

```
**EN:** Configuration for sparse attention. Declares the `SparseConfig` class.
**CN:** 声明 `SparseConfig` 类。

### Lines 58-67: Shared state definitions / 共享状态定义
```python
    top_k: int = 2048
    device_buffer_size: int = 4096
    host_to_device_ratio: int = 2
    algorithm: Optional[str] = None
    backend: Optional[str] = None
    page_size: Optional[int] = None
    min_sparse_prompt_len: Optional[int] = None
    sparse_extra_config: dict = field(
        default_factory=dict
    )  # Algorithm-specific config, parsed by each algorithm
```
**EN:** Defines class-level variables such as `top_k`, `device_buffer_size`, `host_to_device_ratio`, `algorithm`, `backend`.
**CN:** 定义类级变量，例如 `top_k`, `device_buffer_size`, `host_to_device_ratio`, `algorithm`, `backend`。

### Lines 70-95: SparseCoordinator declaration / SparseCoordinator 声明
```python
class SparseCoordinator:
    """
    Coordinator for sparse attention with retrievable KV cache compression.

    This coordinator framework is designed for decode-phase retrievable algorithms
    (e.g., Quest, PQCache, SnapKV) that dynamically select important KV cache entries
    based on current queries. It manages the lifecycle of sparse attention including
    representation construction, sparse retrieval, and token offloading.

    Request Lifecycle and API Calls:
        1. Request Start:
           - on_request_begin(req) -> Register request and initialize state
# ... omitted for brevity ...
        4. Request End:
           - on_request_end(req) -> Clean up state and resources
    """

```
**EN:** Coordinator for sparse attention with retrievable KV cache compression. Declares the `SparseCoordinator` class.
**CN:** 声明 `SparseCoordinator` 类。

### Lines 96-136: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        config: SparseConfig,
        algorithm: BaseSparseAlgorithm,
        backend_adaptor: Optional[BackendAdaptor],
        req_to_token_pool: ReqToTokenPool,
        token_to_kv_pool: KVCache,
        start_layer: int,
        end_layer: int,
        device: torch.device,
    ):
        self.config = config
# ... omitted for brevity ...

        logger.info(
            f"SparseCoordinator initialized with sparse algorithm={type(algorithm).__name__}"
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SparseCoordinator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SparseCoordinator`。

### Lines 138-145: on_request_begin implementation / on_request_begin 实现
```python
    def on_request_begin(self, req: "Req") -> None:
        """
        Handle request begin event. Called when a new request is created.

        Registers the request in the state tracker to enable sparse attention processing.
        """
        if req.req_pool_idx is not None:
            self.states.register(req.req_pool_idx, len(req.origin_input_ids))
```
**EN:** Handle request begin event. Implements the ON request begin routine for this scope. It belongs to `SparseCoordinator`.
**CN:** 实现当前作用域中的on请求begin例程。 该方法属于 `SparseCoordinator`。

### Lines 147-155: on_request_end implementation / on_request_end 实现
```python
    def on_request_end(self, req: "Req") -> None:
        """
        Handle request end event. Called when a request is completed or aborted.
        Cleans up request-specific state and releases resources.
        """
        if req.req_pool_idx is None:
            return

        self.states.clear(req.req_pool_idx)
```
**EN:** Handle request end event. Implements the ON request END routine for this scope. It belongs to `SparseCoordinator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的on请求end例程。 该方法属于 `SparseCoordinator`。它会向调用方返回计算结果。

### Lines 156-159: Comment block / 注释块
```python

        # TODO: Implement request end handling
        # - Release host indices if any were allocated for offloading

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 160-169: forward_begin implementation / forward_begin 实现
```python
    def forward_begin(self, forward_batch: "ForwardBatch") -> None:
        """
        Handle forward pass begin event. Called before each forward pass starts.

        Wait for pending KVCache offloading operations to complete before forward pass.
        Ensures memory consistency for subsequent sparse attention operations.
        """
        # TODO: Implement forward begin handling
        # - Check if there are pending offloading operations
        pass
```
**EN:** Handle forward pass begin event. Implements the forward begin routine for this scope. It belongs to `SparseCoordinator`.
**CN:** 实现当前作用域中的forward begin例程。 该方法属于 `SparseCoordinator`。

### Lines 171-180: forward_end implementation / forward_end 实现
```python
    def forward_end(self, forward_batch: "ForwardBatch") -> None:
        """
        Handle forward pass end event. Called after each forward pass completes.

        Trigger async KVCache offloading operations.
        """
        # TODO: Implement forward end handling
        # - Identify tokens to offload
        # - Trigger async offloading operations
        pass
```
**EN:** Handle forward pass end event. Implements the forward END routine for this scope. It belongs to `SparseCoordinator`.
**CN:** 实现当前作用域中的forward end例程。 该方法属于 `SparseCoordinator`。

### Lines 182-203: attention_begin implementation / attention_begin 实现
```python
    def attention_begin(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: "ForwardBatch",
        attn_metadata: Optional[Any],
        **kwargs,
    ) -> Optional[Any]:
        """
        Handle attention begin event. Called before each attention pass starts.
# ... omitted for brevity ...

        return self._handle_sparse_retrieve(
            query, layer, forward_batch, attn_metadata, **kwargs
        )
```
**EN:** Handle attention begin event. Implements the attention begin routine for this scope. It belongs to `SparseCoordinator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的attention begin例程。 该方法属于 `SparseCoordinator`。它会向调用方返回计算结果。

### Lines 205-234: attention_end implementation / attention_end 实现
```python
    def attention_end(
        self,
        output: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: "ForwardBatch",
    ) -> None:
        """
        Handle attention end event. Called after each attention pass completes.

        Maybe construct and update sparse representations.
        """
        layer_id = layer.layer_id
# ... omitted for brevity ...
            seq_lens=forward_batch.seq_lens,
            k_buffer=self.token_to_kv_pool.get_key_buffer(layer_id),
            forward_batch=forward_batch,
        )
```
**EN:** Handle attention end event. Implements the attention END routine for this scope. It belongs to `SparseCoordinator`.
**CN:** 实现当前作用域中的attention end例程。 该方法属于 `SparseCoordinator`。

### Lines 236-267: _handle_sparse_retrieve implementation / _handle_sparse_retrieve 实现
```python
    def _handle_sparse_retrieve(
        self,
        query: torch.Tensor,
        layer: "RadixAttention",
        forward_batch: "ForwardBatch",
        attn_metadata: Optional[Any],
        **kwargs,
    ) -> Optional[torch.Tensor]:
        req_pool_indices = forward_batch.req_pool_indices
        # Compute Topk
        sparse_mask = self._compute_sparse_mask(req_pool_indices)
        selected_indices, valid_lengths = self.algorithm.retrieve_topk(
# ... omitted for brevity ...
            req_to_token=self.req_to_token_pool.req_to_token,
            page_size=self.page_size,
            layer_id=layer.layer_id,
        )
```
**EN:** Implements the handle sparse retrieve routine for this scope. It belongs to `SparseCoordinator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的handle稀疏retrieve例程。 该方法属于 `SparseCoordinator`。它会向调用方返回计算结果。

### Lines 269-275: _compute_sparse_mask implementation / _compute_sparse_mask 实现
```python
    def _compute_sparse_mask(self, req_pool_indices):
        mask = (
            self.states.prompt_lens[req_pool_indices]
            >= self.config.min_sparse_prompt_len
        )

        return mask
```
**EN:** Implements the compute sparse mask routine for this scope. It belongs to `SparseCoordinator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的compute稀疏mask例程。 该方法属于 `SparseCoordinator`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`RequestTrackers`**: Defines the `RequestTrackers` type and its core responsibilities. / 定义 `RequestTrackers` 类型及其核心职责。
- **`SparseConfig`**: Defines the `SparseConfig` type and its core responsibilities. / 定义 `SparseConfig` 类型及其核心职责。
- **`SparseCoordinator`**: Defines the `SparseCoordinator` type and its core responsibilities. / 定义 `SparseCoordinator` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `dataclasses`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm`, `sglang.srt.mem_cache.sparsity.backend.backend_adaptor`, `sglang.srt.layers.radix_attention`, `sglang.srt.managers.schedule_batch`, `sglang.srt.model_executor.forward_batch_info`
