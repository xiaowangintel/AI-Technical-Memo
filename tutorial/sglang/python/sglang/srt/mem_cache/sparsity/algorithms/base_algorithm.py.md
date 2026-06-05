# base_algorithm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/sparsity/algorithms/base_algorithm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the base algorithm logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的基础算法相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and setup / 导入与初始化
```python
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

import torch
```
**EN:** Imports `abc`, `typing`, `torch` and other helpers used by the surrounding scope.
**CN:** 导入 `abc`, `typing`, `torch` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 6-7: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 10-25: BaseSparseAlgorithm declaration / BaseSparseAlgorithm 声明
```python
class BaseSparseAlgorithm(ABC):
    """
    Abstract base class for sparse attention algorithms.

    This class provides a unified interface for implementing various retrievable KVCache
    compression algorithms. Token-wise sparsity is treated as page-wise with page_size=1.

    References:
        - ChunkKV: https://arxiv.org/abs/2502.00299
        - Quest: https://arxiv.org/pdf/2406.10774
        - PQCache: https://arxiv.org/abs/2407.12820
        - SnapKV: https://arxiv.org/pdf/2404.14469
        - Look-ahead QCache: https://arxiv.org/pdf/2505.20334
        - and more...
    """

```
**EN:** Abstract base class for sparse attention algorithms. Declares the `BaseSparseAlgorithm` class and connects it to `ABC`.
**CN:** 声明 `BaseSparseAlgorithm` 类，并将其关联到 `ABC`。

### Lines 26-30: __init__ implementation / __init__ 实现
```python
    def __init__(self, config, device: torch.device, **kwargs):
        self.config = config
        self.device = device
        self.req_to_token_pool = None
        self.states = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `BaseSparseAlgorithm`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `BaseSparseAlgorithm`。

### Lines 32-53: initialize_representation_pool implementation / initialize_representation_pool 实现
```python
    def initialize_representation_pool(
        self,
        start_layer: int,
        end_layer: int,
        token_to_kv_pool,
        req_to_token_pool,
        states,
    ):
        """
        Initialize algorithm-specific representation pool and set context.

        Called once during SparseCoordinator initialization. Algorithms allocate
# ... omitted for brevity ...
            - SnapKV: Allocate voting scores [num_tokens] and selected positions mask for retention strategy
            - Look-ahead QCache: Allocate importance scores [num_tokens], eviction mask, and optional pseudo query cache [cache_size, hidden_dim]
        """
        pass
```
**EN:** Initialize algorithm-specific representation pool and set context. Implements the initialize representation pool routine for this scope. It belongs to `BaseSparseAlgorithm`.
**CN:** 实现当前作用域中的initialize representation池例程。 该方法属于 `BaseSparseAlgorithm`。

### Lines 55-77: construct_representations implementation / construct_representations 实现
```python
    def construct_representations(
        self,
        layer_id: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        k_buffer: torch.Tensor,
        forward_batch: "ForwardBatch",
    ):
        """
        Construct initial representations during prefill phase.

        Called at every layer during forward pass. Algorithm internally decides
# ... omitted for brevity ...
            - SnapKV: Select observation window (recent tokens), compute attention weights, aggregate via voting to identify important prefix positions, apply 1D pooling to preserve context
            - Look-ahead QCache: Generate pseudo lookahead query (e.g., mean of last k queries), compute KV importance scores, mark low-importance KVs for eviction
        """
        pass
```
**EN:** Construct initial representations during prefill phase. Implements the construct representations routine for this scope. It belongs to `BaseSparseAlgorithm`.
**CN:** 实现当前作用域中的construct representations例程。 该方法属于 `BaseSparseAlgorithm`。

### Lines 79-103: update_representations implementation / update_representations 实现
```python
    def update_representations(
        self,
        layer_id: int,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        k_buffer: torch.Tensor,
        forward_batch: "ForwardBatch",
    ):
        """
        Incrementally update representations during decode phase.

        Called at every layer during forward pass. Algorithm internally decides
# ... omitted for brevity ...
            - SnapKV: Optional: periodically re-run voting with sliding observation window (typically static after prefill)
            - Look-ahead QCache: Periodically regenerate pseudo queries and re-evaluate importance scores to adapt to generation dynamics
        """
        pass
```
**EN:** Incrementally update representations during decode phase. Updates existing state to reflect new inputs. It belongs to `BaseSparseAlgorithm`.
**CN:** 根据新输入更新已有状态。 该方法属于 `BaseSparseAlgorithm`。

### Lines 105-143: retrieve_topk implementation / retrieve_topk 实现
```python
    @abstractmethod
    def retrieve_topk(
        self,
        queries: torch.Tensor,
        layer_id: int,
        req_pool_indices: torch.Tensor,
        sparse_mask: torch.Tensor,
        **kwargs,
    ) -> tuple:
        """
        Retrieve top-k important KV indices for sparse attention.

# ... omitted for brevity ...
            - SnapKV: Return union of voted important prefix positions (with clustered neighbors) and observation window tokens
            - Look-ahead QCache: Return KVs not marked for eviction (eviction based on pseudo query importance evaluation)
        """
        pass
```
**EN:** Retrieve top-k important KV indices for sparse attention. Implements the retrieve topk routine for this scope. It belongs to `BaseSparseAlgorithm`.
**CN:** 实现当前作用域中的retrieve topk例程。 该方法属于 `BaseSparseAlgorithm`。

### Lines 146-162: BaseSparseAlgorithmImpl declaration / BaseSparseAlgorithmImpl 声明
```python
class BaseSparseAlgorithmImpl(BaseSparseAlgorithm):
    """
    Implementation base class for sparse attention algorithms.

    Provides common infrastructure for algorithms that operate at page/chunk granularity
    (token-wise is simply page_size=1):
    - Generic construct/update flow with state tracking
    - TopK retrieval with recent page retention (can be overridden)

    Subclasses need to implement:
    - _initialize_representation_pools(): Initialize algorithm-specific representation pools
    - _compute_page_representations(): Compute page scores/representations
    - _retrieve_page_scores(): Retrieve page scores for TopK selection

    Subclasses can also override any method for specialized behavior
    """

```
**EN:** Implementation base class for sparse attention algorithms. Declares the `BaseSparseAlgorithmImpl` class and connects it to `BaseSparseAlgorithm`.
**CN:** 声明 `BaseSparseAlgorithmImpl` 类，并将其关联到 `BaseSparseAlgorithm`。

### Lines 163-167: __init__ implementation / __init__ 实现
```python
    def __init__(self, config, device: torch.device, **kwargs):
        super().__init__(config, device, **kwargs)
        self.sparsity_ratio = config.sparse_extra_config.get("sparsity_ratio", 0.7)
        self.num_recent_pages = config.sparse_extra_config.get("num_recent_pages", 4)
        self.page_size = config.page_size
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `BaseSparseAlgorithmImpl`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `BaseSparseAlgorithmImpl`。

### Lines 169-187: initialize_representation_pool implementation / initialize_representation_pool 实现
```python
    def initialize_representation_pool(
        self,
        start_layer: int,
        end_layer: int,
        token_to_kv_pool,
        req_to_token_pool,
        states,
    ):
        self.req_to_token_pool = req_to_token_pool
        self.token_to_kv_pool = token_to_kv_pool
        self.start_layer = start_layer
        self.end_layer = end_layer
# ... omitted for brevity ...
        total_num_pages = (total_num_tokens + self.page_size - 1) // self.page_size

        # Initialize algorithm-specific representation pools
        self._initialize_representation_pools(start_layer, end_layer, total_num_pages)
```
**EN:** Implements the initialize representation pool routine for this scope. It belongs to `BaseSparseAlgorithmImpl`.
**CN:** 实现当前作用域中的initialize representation池例程。 该方法属于 `BaseSparseAlgorithmImpl`。

### Lines 189-225: construct_representations implementation / construct_representations 实现
```python
    def construct_representations(
        self,
        layer_id,
        req_pool_indices,
        seq_lens,
        k_buffer,
        forward_batch,
    ) -> torch.Tensor:

        if not forward_batch.forward_mode.is_extend():
            return

# ... omitted for brevity ...
        if layer_id == self.end_layer - 1:
            success_indices = req_pool_indices[valid_mask]
            self.states.repr_constructed[success_indices] = True
            self.states.last_constructed_page[success_indices] = num_pages[valid_mask]
```
**EN:** Implements the construct representations routine for this scope. It belongs to `BaseSparseAlgorithmImpl`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的construct representations例程。 该方法属于 `BaseSparseAlgorithmImpl`。它会向调用方返回计算结果。

### Lines 227-260: update_representations implementation / update_representations 实现
```python
    def update_representations(
        self,
        layer_id,
        req_pool_indices,
        seq_lens,
        k_buffer,
        forward_batch,
    ) -> torch.Tensor:
        if not forward_batch.forward_mode.is_decode_or_idle():
            return

        start_page = self.states.last_constructed_page[req_pool_indices]
# ... omitted for brevity ...
        # Update tracking states
        if layer_id == self.end_layer - 1:
            success_indices = req_pool_indices[valid_mask]
            self.states.last_constructed_page[success_indices] = end_page[valid_mask]
```
**EN:** Updates existing state to reflect new inputs. It belongs to `BaseSparseAlgorithmImpl`. It returns a computed result to its caller.
**CN:** 根据新输入更新已有状态。 该方法属于 `BaseSparseAlgorithmImpl`。它会向调用方返回计算结果。

### Lines 262-355: retrieve_topk implementation / retrieve_topk 实现
```python
    def retrieve_topk(
        self,
        queries: torch.Tensor,
        layer_id: int,
        req_pool_indices: torch.Tensor,
        sparse_mask: torch.Tensor,
        **kwargs,
    ) -> tuple:
        """
        Default TopK retrieval: score-based selection + recent pages.
        Subclasses can override for query-dependent retrieval.

# ... omitted for brevity ...
            out_indices[i, :length] = selected
            out_lengths[i] = length

        return out_indices, out_lengths
```
**EN:** Default TopK retrieval: score-based selection + recent pages. Implements the retrieve topk routine for this scope. It belongs to `BaseSparseAlgorithmImpl`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的retrieve topk例程。 该方法属于 `BaseSparseAlgorithmImpl`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 357-361: _initialize_representation_pools implementation / _initialize_representation_pools 实现
```python
    def _initialize_representation_pools(
        self, start_layer: int, end_layer: int, total_num_pages: int
    ):
        """Initialize algorithm-specific representation pools for all layers."""
        raise NotImplementedError
```
**EN:** Initialize algorithm-specific representation pools for all layers. Implements the initialize representation pools routine for this scope. It belongs to `BaseSparseAlgorithmImpl`. It validates error cases explicitly.
**CN:** 实现当前作用域中的initialize representation pools例程。 该方法属于 `BaseSparseAlgorithmImpl`。它会显式处理错误场景。

### Lines 363-373: _compute_page_representations implementation / _compute_page_representations 实现
```python
    def _compute_page_representations(
        self,
        layer_id: int,
        reqs: torch.Tensor,
        seq_lens: torch.Tensor,
        start_page,
        end_page: torch.Tensor,
        k_buffer: torch.Tensor,
    ):
        """Compute and store page representations for given page range."""
        raise NotImplementedError
```
**EN:** Compute and store page representations for given page range. Implements the compute page representations routine for this scope. It belongs to `BaseSparseAlgorithmImpl`. It validates error cases explicitly.
**CN:** 实现当前作用域中的compute页representations例程。 该方法属于 `BaseSparseAlgorithmImpl`。它会显式处理错误场景。

### Lines 375-383: _retrieve_page_scores implementation / _retrieve_page_scores 实现
```python
    def _retrieve_page_scores(
        self,
        layer_id: int,
        phys_pages: torch.Tensor,
        req_pool_indices: torch.Tensor,
        queries: torch.Tensor,
    ) -> torch.Tensor:
        """Retrieve page scores for TopK selection."""
        raise NotImplementedError
```
**EN:** Retrieve page scores for TopK selection. Implements the retrieve page scores routine for this scope. It belongs to `BaseSparseAlgorithmImpl`. It validates error cases explicitly.
**CN:** 实现当前作用域中的retrieve页scores例程。 该方法属于 `BaseSparseAlgorithmImpl`。它会显式处理错误场景。

## Key Concepts / 关键概念
- **`BaseSparseAlgorithm`**: Defines the `BaseSparseAlgorithm` type and its core responsibilities. / 定义 `BaseSparseAlgorithm` 类型及其核心职责。
- **`BaseSparseAlgorithmImpl`**: Defines the `BaseSparseAlgorithmImpl` type and its core responsibilities. / 定义 `BaseSparseAlgorithmImpl` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.model_executor.forward_batch_info`
