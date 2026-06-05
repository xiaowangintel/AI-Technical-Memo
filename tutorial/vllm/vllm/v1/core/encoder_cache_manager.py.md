# encoder_cache_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/encoder_cache_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EncoderCacheManager`, `compute_mm_encoder_budget`, `EncoderDecoderCacheManager` for the V1 `core` subsystem. / 为 V1 的 `core` 子系统实现 `EncoderCacheManager`, `compute_mm_encoder_budget`, `EncoderDecoderCacheManager`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections import OrderedDict
from collections.abc import Mapping
from typing import TYPE_CHECKING

from vllm.logger import init_logger
from vllm.v1.request import Request

if TYPE_CHECKING:
    from vllm.config import SchedulerConfig

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `EncoderCacheManager` class / `EncoderCacheManager` 类
```python
class EncoderCacheManager:
    """Manages caching of encoder outputs for multimodal models in vLLM V1.

    The EncoderCacheManager handles the lifecycle of multimodal encoder outputs
    (such as vision embeddings from images) during request processing. It
    provides memory-aware caching to avoid recomputing encoder outputs when the
    same multimodal inputs appear in different stages of request processing.

    This manager is particularly important for:
    - Vision-language models (e.g., LLaVA) where image encoder outputs are
      cached
    - Any multimodal model where encoder computation is expensive and
      cacheable

    The cache operates at the granularity of individual multimodal input items
    within requests, allowing for fine-grained memory management and enabling
    chunked processing of multimodal inputs.

    Cache is enabled to share embeddings of same multimodal data
    item (identified by their hash value) between different requests,
    and eviction takes place at allocation time when there's no free
    space for new embeddings.
    Oldest cached embeddings with no request referenced will be first evicted.

    NOTE: The EncoderCacheManager operates on the level of multimodal embeddings
    instead of encoder tokens (i.e. all tokens that represent the multimodal data
    in the input sequence). This means all break/text tokens in-between multimodal
    embeddings are not considered with respect to the cache size and the number
    of free slots.

    Args:
        cache_size: Limit the size of the cache, measured by the number of
                    encoder embeddings from the input sequence.

    Attributes:
        cache_size: Total cache capacity in encoder embeddings.
        num_free_slots: Current available cache capacity in encoder embeddings.
        num_freeable_slots: Capacity that can be immediately reclaimed by
            evicting entries with zero references (in encoder embeddings).
        cached: Mapping from mm_hash to a set of request IDs that currently
            reference the cached entry. If the set is empty, the entry exists
            but is not referenced by any request and is eligible for
            reclamation.
        freeable: List of tuples (mm_hash, num_encoder_embeds) representing entries
            whose no current running request is needed and that can be freed to
            make space when needed.
        freed: List of mm_hash strings that were actually evicted since the
            last call to get_freed_mm_hashes(). This list is cleared on return.
    """
```
**EN:** Introduces the `EncoderCacheManager` class. Core methods include `__init__`, `reset`, `check_and_update_cache`, `can_allocate`, `allocate`, `get_cached_input_ids`. Docstring signal: Manages caching of encoder outputs for multimodal models in vLLM V1.
**CN:** 这里定义 `EncoderCacheManager` 类。核心方法包括 `__init__`, `reset`, `check_and_update_cache`, `can_allocate`, `allocate`, `get_cached_input_ids`。

### `EncoderCacheManager.__init__` method / `EncoderCacheManager.__init__` 方法
```python
    def __init__(self, cache_size: int):
        self.cache_size = cache_size
        self.num_free_slots = cache_size
        self.num_freeable_slots = cache_size

        # mm_hash of mm_data => ids of requests that reference the mm_data
        self.cached: dict[str, set[str]] = {}

        # mm_hash of mm_data => num_encoder_embeds of the mm_data
        self.freeable: OrderedDict[str, int] = OrderedDict()
        self.freed: list[str] = []
```
**EN:** This method initializes the object state within `EncoderCacheManager`. Key calls include `OrderedDict`. It touches state such as `cache_size`, `num_free_slots`, `num_freeable_slots`, `cached`, `freeable`, `freed`.
**CN:** 该方法会初始化对象状态，其作用域位于`EncoderCacheManager`。 关键调用包括 `OrderedDict`。 它会读写 `cache_size`, `num_free_slots`, `num_freeable_slots`, `cached`, `freeable`, `freed` 等状态。

### `EncoderCacheManager.check_and_update_cache` method / `EncoderCacheManager.check_and_update_cache` 方法
```python
    def check_and_update_cache(self, request: Request, input_id: int) -> bool:
        """Check if encoder output for a specific multimodal input is cached.

        If the encoder output is cached, update `cached` to add the request id
        to the set of request ids that reference the cached encoder output.
        If the encoder output was previously not referenced by any request,
        update `freeable` and `num_freeable_slots` accordingly.

        Args:
            request: The request containing the multimodal input
            input_id: Index of the multimodal input within the request

        Returns:
            True if the encoder output for this input is already cached
        """
        mm_hash = request.mm_features[input_id].identifier
        # Not cached at all
        if mm_hash not in self.cached:
            return False

        # Cached but currently not referenced by any request
        if not self.cached[mm_hash]:
            num_encoder_embeds = self.freeable.pop(mm_hash)
            self.num_freeable_slots -= num_encoder_embeds

        self.cached[mm_hash].add(request.request_id)
        return True
```
**EN:** This method validates assumptions or constraints within `EncoderCacheManager`. The docstring frames it as: Check if encoder output for a specific multimodal input is cached. Key calls include `add`, `pop`. It touches state such as `num_freeable_slots`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`EncoderCacheManager`。 关键调用包括 `add`, `pop`。 它会读写 `num_freeable_slots` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderCacheManager.can_allocate` method / `EncoderCacheManager.can_allocate` 方法
```python
    def can_allocate(
        self,
        request: Request,
        input_id: int,
        encoder_compute_budget: int,
        num_embeds_to_schedule: int,
    ) -> bool:
        """Check if there's sufficient cache space for a multimodal input.
        If there is, return True and update EncoderCacheManager state.

        If there is not enough free space in `num_free_slots` but there is
        enough reclaimable space in `num_freeable_slots`, entries will be
        evicted from `freeable` (their mm_hash appended to `freed`) until
        enough space is available, and then this method returns True.
        Older entries are evicted first.

        Returns False only if the requested number of tokens exceeds both
        the free and reclaimable capacities combined.

        Args:
            request: The request containing the multimodal input.
            input_id: Index of the multimodal input within the request.
            encoder_compute_budget: Number of encoder embeddings allowed to be
                computed when this method is invoked.
            num_embeds_to_schedule: Number of encoder embeddings already scheduled to be
                allocated with cache space when this method is invoked.

        Returns:
            True if there's enough capacity to hold the encoder output for this
            input (possibly after reclaiming `freeable` entries); otherwise
            False.

        Note: This method does not allocate physical memory for the encoder
        output but only the state of EncoderCacheManager.
        """
        num_embeds = request.get_num_encoder_embeds(input_id)

        # Not enough compute budget
        if num_embeds > encoder_compute_budget:
            return False

        num_embeds += num_embeds_to_schedule

        # Enough free slots
        if num_embeds <= self.num_free_slots:
            return True

        # Not enough reclaimable slots
        if num_embeds > self.num_freeable_slots:
            return False

        # Not enough free slots but enough reclaimable slots
        # NOTE: Eviction takes place here, but physical memory is not freed
        # until model runner is notified by the scheduler output.
        while num_embeds > self.num_free_slots:
            mm_hash, num_free_embeds = self.freeable.popitem(last=False)
            del self.cached[mm_hash]
            self.freed.append(mm_hash)
            self.num_free_slots += num_free_embeds
        return True
```
**EN:** This method implements `can_allocate` within `EncoderCacheManager`. The docstring frames it as: Check if there's sufficient cache space for a multimodal input. Key calls include `get_num_encoder_embeds`, `popitem`, `append`. It touches state such as `num_free_slots`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `can_allocate`，其作用域位于`EncoderCacheManager`。 关键调用包括 `get_num_encoder_embeds`, `popitem`, `append`。 它会读写 `num_free_slots` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `EncoderCacheManager.allocate` method / `EncoderCacheManager.allocate` 方法
```python
    def allocate(self, request: Request, input_id: int) -> None:
        """Allocate cache space for a multimodal input's encoder output.

        This reserves cache space for storing the encoder output of the
        specified multimodal input. The actual encoder output storage happens in
        the model runner; this method updates the manager's bookkeeping.

        Note:
            This method assumes can_allocate() returned True for the same input.
        """

        mm_hash = request.mm_features[input_id].identifier
        request_id = request.request_id
        if mm_hash not in self.cached:
            self.cached[mm_hash] = set()

        num_encoder_embeds = request.get_num_encoder_embeds(input_id)

        # NOTE: Encoder cache should always have enough space for encoder inputs
        # that are scheduled since eviction takes place at can_allocate().
        assert self.num_free_slots >= num_encoder_embeds
        assert self.num_freeable_slots >= num_encoder_embeds

        self.cached[mm_hash].add(request_id)
        self.num_free_slots -= num_encoder_embeds
        self.num_freeable_slots -= num_encoder_embeds
```
**EN:** This method implements `allocate` within `EncoderCacheManager`. The docstring frames it as: Allocate cache space for a multimodal input's encoder output. Key calls include `get_num_encoder_embeds`, `add`, `set`. It touches state such as `num_free_slots`, `num_freeable_slots`, `cached`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `allocate`，其作用域位于`EncoderCacheManager`。 关键调用包括 `get_num_encoder_embeds`, `add`, `set`。 它会读写 `num_free_slots`, `num_freeable_slots`, `cached` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderCacheManager.get_cached_input_ids` method / `EncoderCacheManager.get_cached_input_ids` 方法
```python
    def get_cached_input_ids(self, request: Request) -> set[int]:
        """Get all cached multimodal input IDs for a request.

        Returns the set of input IDs whose `mm_hash` exists in the cache map.
        This includes entries that are currently unreferenced (and thus present
        in `freeable`); for such entries, freeing for this request will be a
        no-op.
        """
        return {
            input_id
            for input_id in range(len(request.mm_features))
            if request.mm_features[input_id].identifier in self.cached
        }
```
**EN:** This method returns or derives a value within `EncoderCacheManager`. The docstring frames it as: Get all cached multimodal input IDs for a request. Key calls include `range`, `len`.
**CN:** 该方法会返回或推导一个值，其作用域位于`EncoderCacheManager`。 关键调用包括 `range`, `len`。

### `EncoderCacheManager.free_encoder_input` method / `EncoderCacheManager.free_encoder_input` 方法
```python
    def free_encoder_input(self, request: Request, input_id: int) -> None:
        """Free the request's reference to the encoder input (`mm_data`)

        When the reference set for the corresponding `mm_hash` becomes empty,
        the entry is appended to `freeable` and `num_freeable_slots` is
        increased by the number of encoder embeddings for that input.

        The entry is NOT physically freed until capacity is needed (e.g., by
        `can_allocate`).
        """
        req_id = request.request_id
        mm_hash = request.mm_features[input_id].identifier
        # The mm_hash not in cache or the req_id set is empty
        if not self.cached.get(mm_hash, None):
            return
        self.cached[mm_hash].discard(req_id)
        if not self.cached[mm_hash]:
            num_encoder_embeds = request.get_num_encoder_embeds(input_id)
            self.freeable[mm_hash] = num_encoder_embeds
            self.num_freeable_slots += num_encoder_embeds
```
**EN:** This method implements `free_encoder_input` within `EncoderCacheManager`. The docstring frames it as: Free the request's reference to the encoder input (`mm_data`) When the reference set for the corresponding `mm_hash` becomes empty, the entry is appended to `freeable` and `num_freeable_slots` is increased by the number of encoder embeddings for that input. Key calls include `discard`, `get`, `get_num_encoder_embeds`. It touches state such as `freeable`, `num_freeable_slots`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `free_encoder_input`，其作用域位于`EncoderCacheManager`。 关键调用包括 `discard`, `get`, `get_num_encoder_embeds`。 它会读写 `freeable`, `num_freeable_slots` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderCacheManager.free` method / `EncoderCacheManager.free` 方法
```python
    def free(self, request: Request) -> None:
        """Free all encoder input cache reference held by *request*.

        For each cached input ID, `free_encoder_input` is invoked.
        The data stays in memory until eviction is triggered by a future
        attempt allocation called by 'can_allocate'.

        Typically called when a request is finished, cancelled, or aborted.
        """
        input_ids = self.get_cached_input_ids(request)
        for input_id in input_ids:
            self.free_encoder_input(request, input_id)
```
**EN:** This method implements `free` within `EncoderCacheManager`. The docstring frames it as: Free all encoder input cache reference held by *request*. Key calls include `get_cached_input_ids`, `free_encoder_input`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `free`，其作用域位于`EncoderCacheManager`。 关键调用包括 `get_cached_input_ids`, `free_encoder_input`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `EncoderCacheManager.get_freed_mm_hashes` method / `EncoderCacheManager.get_freed_mm_hashes` 方法
```python
    def get_freed_mm_hashes(self) -> list[str]:
        """Get and clear the list of recently freed encoder cache entries.

        Returns:
            List of mm_hash strings that were actually evicted since the last
            call to be used by the scheduler to notify workers about which
            encoder outputs can be removed from their caches. The internal
            list is cleared after this call.
        """
        freed = self.freed
        self.freed = []
        return freed
```
**EN:** This method returns or derives a value within `EncoderCacheManager`. The docstring frames it as: Get and clear the list of recently freed encoder cache entries. It touches state such as `freed`.
**CN:** 该方法会返回或推导一个值，其作用域位于`EncoderCacheManager`。 它会读写 `freed` 等状态。

### `compute_mm_encoder_budget` function / `compute_mm_encoder_budget` 函数
```python
def compute_mm_encoder_budget(
    scheduler_config: "SchedulerConfig",
    mm_max_toks_per_item: Mapping[str, int],
) -> tuple[int, int]:
    """Compute the encoder cache budget based on the model and scheduler
    configurations for a multimodal model.

    Args:
        scheduler_config: Scheduler configuration.
        mm_max_toks_per_item: The maximum number of tokens per item for each
            non-text modality.

    Returns:
        - Compute budget for encoder execution, measured in number of tokens
            from the input sequence.
        - Space budget for encoder cache size, measured in number of tokens
            from the input sequence.
    """

    if not mm_max_toks_per_item:
        logger.warning(
            "All non-text modalities supported by the model have been "
            "explicitly disabled via limit_mm_per_prompt. Encoder cache will "
            "not be initialized."
        )
        return 0, 0

    max_tokens_per_mm_item = max(mm_max_toks_per_item.values())

    if (
        scheduler_config.disable_chunked_mm_input
        and max_tokens_per_mm_item > scheduler_config.max_num_batched_tokens
    ):
        raise ValueError(
            "Chunked MM input disabled but max_tokens_per_mm_item "
            f"({max_tokens_per_mm_item}) is larger than max_num_batched_tokens"
            f" ({scheduler_config.max_num_batched_tokens}). Please increase "
            "max_num_batched_tokens."
        )

    encoder_compute_budget = max(
        scheduler_config.max_num_encoder_input_tokens, max_tokens_per_mm_item
    )
    encoder_cache_size = max(
        scheduler_config.encoder_cache_size, max_tokens_per_mm_item
    )

    return encoder_compute_budget, encoder_cache_size
```
**EN:** This function computes derived values within the module. The docstring frames it as: Compute the encoder cache budget based on the model and scheduler configurations for a multimodal model. Key calls include `max`, `warning`, `values`, `ValueError`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `max`, `warning`, `values`, `ValueError`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderDecoderCacheManager` class / `EncoderDecoderCacheManager` 类
```python
class EncoderDecoderCacheManager(EncoderCacheManager):
```
**EN:** Introduces the `EncoderDecoderCacheManager` class on top of `EncoderCacheManager`. Core methods include `__init__`, `reset`, `check_and_update_cache`, `can_allocate`, `allocate`, `free`.
**CN:** 这里定义 `EncoderDecoderCacheManager` 类，其基类包括 `EncoderCacheManager`。核心方法包括 `__init__`, `reset`, `check_and_update_cache`, `can_allocate`, `allocate`, `free`。

### `EncoderDecoderCacheManager.__init__` method / `EncoderDecoderCacheManager.__init__` 方法
```python
    def __init__(self, cache_size: int):
        self.cache_size = cache_size
        self.num_free_slots = cache_size
        self.allocated: list[str] = []
        self.to_free: list[str] = []
```
**EN:** This method initializes the object state within `EncoderDecoderCacheManager`. It touches state such as `cache_size`, `num_free_slots`, `allocated`, `to_free`.
**CN:** 该方法会初始化对象状态，其作用域位于`EncoderDecoderCacheManager`。 它会读写 `cache_size`, `num_free_slots`, `allocated`, `to_free` 等状态。

### `EncoderDecoderCacheManager.reset` method / `EncoderDecoderCacheManager.reset` 方法
```python
    def reset(self) -> None:
        """Reset the encoder cache to its initial state."""
        self.num_free_slots = self.cache_size
        self.allocated.clear()
        self.to_free.clear()
```
**EN:** This method implements `reset` within `EncoderDecoderCacheManager`. The docstring frames it as: Reset the encoder cache to its initial state. Key calls include `clear`. It touches state such as `num_free_slots`.
**CN:** 该方法会实现 `reset`，其作用域位于`EncoderDecoderCacheManager`。 关键调用包括 `clear`。 它会读写 `num_free_slots` 等状态。

### `EncoderDecoderCacheManager.check_and_update_cache` method / `EncoderDecoderCacheManager.check_and_update_cache` 方法
```python
    def check_and_update_cache(self, request: Request, input_id: int) -> bool:
        return False
```
**EN:** This method validates assumptions or constraints within `EncoderDecoderCacheManager`.
**CN:** 该方法会校验前提与约束，其作用域位于`EncoderDecoderCacheManager`。

### `EncoderDecoderCacheManager.can_allocate` method / `EncoderDecoderCacheManager.can_allocate` 方法
```python
    def can_allocate(
        self,
        request: Request,
        input_id: int,
        encoder_compute_budget: int,
        num_embeds_to_schedule: int,
    ) -> bool:
        num_encoder_embeds = request.get_num_encoder_embeds(input_id)
        # Not enough compute budget
        if num_encoder_embeds > encoder_compute_budget:
            return False

        num_encoder_embeds += num_embeds_to_schedule
        # Enough free slots
        return num_encoder_embeds <= self.num_free_slots
```
**EN:** This method implements `can_allocate` within `EncoderDecoderCacheManager`. Key calls include `get_num_encoder_embeds`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `can_allocate`，其作用域位于`EncoderDecoderCacheManager`。 关键调用包括 `get_num_encoder_embeds`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EncoderDecoderCacheManager.allocate` method / `EncoderDecoderCacheManager.allocate` 方法
```python
    def allocate(self, request: Request, input_id: int) -> None:
        num_encoder_embeds = request.get_num_encoder_embeds(input_id)
        self.num_free_slots -= num_encoder_embeds

        mm_hash = request.mm_features[input_id].identifier
        self.allocated.append(mm_hash)
```
**EN:** This method implements `allocate` within `EncoderDecoderCacheManager`. Key calls include `get_num_encoder_embeds`, `append`. It touches state such as `num_free_slots`.
**CN:** 该方法会实现 `allocate`，其作用域位于`EncoderDecoderCacheManager`。 关键调用包括 `get_num_encoder_embeds`, `append`。 它会读写 `num_free_slots` 等状态。

### `EncoderDecoderCacheManager.free` method / `EncoderDecoderCacheManager.free` 方法
```python
    def free(self, request: Request) -> None:
        for input_id in range(len(request.mm_features)):
            self.free_encoder_input(request, input_id)
```
**EN:** This method implements `free` within `EncoderDecoderCacheManager`. Key calls include `range`, `len`, `free_encoder_input`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `free`，其作用域位于`EncoderDecoderCacheManager`。 关键调用包括 `range`, `len`, `free_encoder_input`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `EncoderDecoderCacheManager.get_freed_mm_hashes` method / `EncoderDecoderCacheManager.get_freed_mm_hashes` 方法
```python
    def get_freed_mm_hashes(self) -> list[str]:
        # As encoder cache is not used for enc-dec models, we can free the entries here
        # The actual free happens in the runner, *before* the model is executed.
        # Therefore, `freeable` acts as a buffer to free the entries only after the
        # model is executed, mimicking the state transition of `EncoderCacheManager`.
        to_free = self.to_free
        self.to_free = self.allocated
        self.allocated = []
        return to_free
```
**EN:** This method returns or derives a value within `EncoderDecoderCacheManager`. It touches state such as `to_free`, `allocated`.
**CN:** 该方法会返回或推导一个值，其作用域位于`EncoderDecoderCacheManager`。 它会读写 `to_free`, `allocated` 等状态。

### `EncoderDecoderCacheManager.free_encoder_input` method / `EncoderDecoderCacheManager.free_encoder_input` 方法
```python
    def free_encoder_input(self, request: Request, input_id: int) -> None:
        num_encoder_embeds = request.get_num_encoder_embeds(input_id)
        self.num_free_slots += num_encoder_embeds
```
**EN:** This method implements `free_encoder_input` within `EncoderDecoderCacheManager`. Key calls include `get_num_encoder_embeds`. It touches state such as `num_free_slots`.
**CN:** 该方法会实现 `free_encoder_input`，其作用域位于`EncoderDecoderCacheManager`。 关键调用包括 `get_num_encoder_embeds`。 它会读写 `num_free_slots` 等状态。

## Key Concepts / 关键概念
- `EncoderCacheManager`: central class or interface in this module. / `EncoderCacheManager`：本模块中的核心类或接口。
- `compute_mm_encoder_budget`: top-level helper or orchestration entry point. / `compute_mm_encoder_budget`：顶层辅助函数或编排入口。
- `EncoderDecoderCacheManager`: central class or interface in this module. / `EncoderDecoderCacheManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.v1.request`, `vllm.config`
