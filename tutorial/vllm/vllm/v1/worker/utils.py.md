# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_zero_kv_blocks_kernel`, `KVBlockZeroer`, `AttentionGroup` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `_zero_kv_blocks_kernel`, `KVBlockZeroer`, `AttentionGroup`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import math
from collections import defaultdict
from collections.abc import Iterable
from dataclasses import dataclass, field
from itertools import product as iprod
from typing import Any

import torch

from vllm.config import CacheConfig, VllmConfig
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.models.interfaces import MultiModalEmbeddings
from vllm.model_executor.models.utils import extract_layer_index
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import largest_power_of_2_divisor
from vllm.utils.mem_utils import MemorySnapshot, format_gib
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadataBuilder,
    MultipleOf,
)
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    EncoderOnlyAttentionSpec,
    FullAttentionSpec,
    KVCacheConfig,
    KVCacheGroupSpec,
    KVCacheSpec,
    MambaSpec,
    UniformTypeKVCacheSpecs,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_zero_kv_blocks_kernel` function / `_zero_kv_blocks_kernel` 函数
```python
@triton.jit
def _zero_kv_blocks_kernel(
    seg_addrs_ptr,
    block_ids_ptr,
    n_blocks,
    N_SEGS: tl.constexpr,
    PAGE_SIZE_EL: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Zero KV cache blocks across all segments in a single launch.

    Each segment is a contiguous region of one block's data.  For backends
    where blocks are outermost (block_dim=0) there is one segment per
    buffer.  For backends where K/V is outermost (block_dim=1) there are
    two segments per buffer (one for K, one for V).

    seg_addrs_ptr holds absolute byte addresses (int64) for each segment,
    allowing segments to live in different CUDA allocations.

    Programs are mapped as (block_index, seg_index, chunk_index).
    """
    pid = tl.program_id(0)
    chunks = PAGE_SIZE_EL // BLOCK_SIZE
    work_per_block = N_SEGS * chunks
    block_index = pid // work_per_block
    if block_index >= n_blocks:
        return
    remainder = pid % work_per_block
    seg_index = remainder // chunks
    chunk_index = remainder % chunks
    block_id = tl.load(block_ids_ptr + block_index)
    seg_addr = tl.load(seg_addrs_ptr + seg_index)
    ptr = tl.cast(seg_addr, tl.pointer_type(tl.int32))
    offset = (
        block_id.to(tl.int64) * PAGE_SIZE_EL + chunk_index.to(tl.int64) * BLOCK_SIZE
    )
    cols = tl.arange(0, BLOCK_SIZE).to(tl.int64)
    tl.store(ptr + offset + cols, tl.zeros([BLOCK_SIZE], dtype=tl.int32))
```
**EN:** This function implements `_zero_kv_blocks_kernel` within the module. The docstring frames it as: Zero KV cache blocks across all segments in a single launch. Key calls include `program_id`, `load`, `cast`, `to`, `store`, `pointer_type`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_zero_kv_blocks_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `cast`, `to`, `store`, `pointer_type`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `KVBlockZeroer` class / `KVBlockZeroer` 类
```python
class KVBlockZeroer:
    """Manages efficient zeroing of KV cache blocks via a Triton kernel.

    Call :meth:`init_meta` once after KV caches are allocated to precompute
    segment addresses, then call :meth:`zero_block_ids` each step to zero
    newly-allocated blocks.
    """
```
**EN:** Introduces the `KVBlockZeroer` class. Core methods include `__init__`, `init_meta`, `zero_block_ids`. Docstring signal: Manages efficient zeroing of KV cache blocks via a Triton kernel.
**CN:** 这里定义 `KVBlockZeroer` 类。核心方法包括 `__init__`, `init_meta`, `zero_block_ids`。

### `KVBlockZeroer.__init__` method / `KVBlockZeroer.__init__` 方法
```python
    def __init__(self, device: torch.device, pin_memory: bool):
        self.device = device
        self.pin_memory = pin_memory
        self._meta: tuple[torch.Tensor, int, int, int] | None = None
        self._id_cap: int = 0
        self._ids_pinned: torch.Tensor | None = None
        self._ids_gpu: torch.Tensor | None = None
```
**EN:** This method initializes the object state within `KVBlockZeroer`. It touches state such as `device`, `pin_memory`, `_meta`, `_id_cap`, `_ids_pinned`, `_ids_gpu`.
**CN:** 该方法会初始化对象状态，其作用域位于`KVBlockZeroer`。 它会读写 `device`, `pin_memory`, `_meta`, `_id_cap`, `_ids_pinned`, `_ids_gpu` 等状态。

### `KVBlockZeroer.init_meta` method / `KVBlockZeroer.init_meta` 方法
```python
    def init_meta(
        self,
        attn_groups_iter: Iterable["AttentionGroup"],
        kernel_block_sizes: list[int],
        cache_dtype: str,
        runner_only_attn_layers: set[str],
        static_forward_context: dict[str, Any],
    ) -> None:
        """One-time precomputation for zero_block_ids.

        Builds absolute-address table for the Triton zeroing kernel.
        Each entry is the absolute byte address of a segment start on the
        GPU, so segments in different CUDA allocations work correctly.

        Block IDs from the scheduler reference logical blocks whose size
        may differ from the kernel block size (virtual block splitting).
        PAGE_SIZE_EL accounts for this ratio so that
        ``block_id * PAGE_SIZE_EL`` lands at the correct offset.

        Only AttentionSpec layers are processed; Mamba layers are skipped.
        """
        seen_ptrs: set[int] = set()
        seg_addrs: list[int] = []
        page_size_el: int | None = None

        for group in attn_groups_iter:
            spec = group.kv_cache_spec
            if not isinstance(spec, FullAttentionSpec):
                continue
            if group.kv_cache_group_id >= len(kernel_block_sizes):
                continue
            kernel_bs = kernel_block_sizes[group.kv_cache_group_id]
            ratio = spec.block_size // kernel_bs
            block_dim = group.backend.get_kv_cache_block_dim(
                kernel_bs,
                spec.num_kv_heads,
                spec.head_size,
                cache_dtype_str=cache_dtype,
            )

            for layer_name in group.layer_names:
                if layer_name in runner_only_attn_layers:
                    continue
                kv = static_forward_context[layer_name].kv_cache
                if not isinstance(kv, torch.Tensor):
                    continue
                dp = kv.data_ptr()
                if dp in seen_ptrs:
                    continue
                seen_ptrs.add(dp)
    # ... omitted for brevity ...
                    if kv.stride(d) * el > block_stride_bytes
                ]
                outer_strides = [kv.stride(d) * el for d in outer_dims]
                for outer in iprod(*(range(kv.shape[d]) for d in outer_dims)):
                    off_bytes = sum(i * s for i, s in zip(outer, outer_strides))
                    seg_addrs.append(dp + off_bytes)

        if not seg_addrs or page_size_el is None:
            self._meta = None
            return

        blk_size = min(largest_power_of_2_divisor(page_size_el), 1024)
        self._id_cap = 8192
        self._ids_pinned = torch.empty(
            self._id_cap,
            dtype=torch.int64,
            pin_memory=self.pin_memory,
        )
        self._ids_gpu = torch.empty(self._id_cap, dtype=torch.int64, device=self.device)
        self._meta = (
            torch.tensor(seg_addrs, dtype=torch.uint64, device=self.device),
            page_size_el,
            blk_size,
            len(seg_addrs),
        )
```
**EN:** This method implements `init_meta` within `KVBlockZeroer`. The docstring frames it as: One-time precomputation for zero_block_ids. Key calls include `set`, `min`, `empty`, `get_kv_cache_block_dim`, `largest_power_of_2_divisor`, `tensor`. It touches state such as `_id_cap`, `_ids_pinned`, `_ids_gpu`, `_meta`. The control flow contains 7 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `init_meta`，其作用域位于`KVBlockZeroer`。 关键调用包括 `set`, `min`, `empty`, `get_kv_cache_block_dim`, `largest_power_of_2_divisor`, `tensor`。 它会读写 `_id_cap`, `_ids_pinned`, `_ids_gpu`, `_meta` 等状态。 控制流包含 7 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `KVBlockZeroer.zero_block_ids` method / `KVBlockZeroer.zero_block_ids` 方法
```python
    def zero_block_ids(self, block_ids: list[int]) -> None:
        """Zero the KV cache memory for the given block IDs."""
        if not block_ids or self._meta is None:
            return
        seg_addrs, page_size_el, blk_size, n_segs = self._meta
        n_blocks = len(block_ids)
        if n_blocks > self._id_cap:
            self._id_cap = n_blocks * 2
            self._ids_pinned = torch.empty(
                self._id_cap,
                dtype=torch.int64,
                pin_memory=self.pin_memory,
            )
            self._ids_gpu = torch.empty(
                self._id_cap, dtype=torch.int64, device=self.device
            )
        assert self._ids_pinned is not None and self._ids_gpu is not None
        self._ids_pinned[:n_blocks].numpy()[:] = block_ids
        idx = self._ids_gpu[:n_blocks]
        idx.copy_(self._ids_pinned[:n_blocks], non_blocking=True)
        grid = (n_blocks * n_segs * (page_size_el // blk_size),)
        _zero_kv_blocks_kernel[grid](
            seg_addrs,
            idx,
            n_blocks,
            N_SEGS=n_segs,
            PAGE_SIZE_EL=page_size_el,
            BLOCK_SIZE=blk_size,
        )
```
**EN:** This method implements `zero_block_ids` within `KVBlockZeroer`. The docstring frames it as: Zero the KV cache memory for the given block IDs. Key calls include `len`, `copy_`, `empty`, `numpy`. It touches state such as `_ids_pinned`, `_id_cap`, `_ids_gpu`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `zero_block_ids`，其作用域位于`KVBlockZeroer`。 关键调用包括 `len`, `copy_`, `empty`, `numpy`。 它会读写 `_ids_pinned`, `_id_cap`, `_ids_gpu` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionGroup` class / `AttentionGroup` 类
```python
@dataclass
class AttentionGroup:
    backend: type[AttentionBackend]
    layer_names: list[str]
    kv_cache_spec: KVCacheSpec
    kv_cache_group_id: int
    # When ubatching is enabled we will have a metadata builder for each ubatch
    # so that if they use internal persistent buffers for cudagraphs, and they
    # won't have to worry about conflicting with the other ubatches.
    metadata_builders: list[AttentionMetadataBuilder] = field(
        default_factory=lambda: []
    )
```
**EN:** Uses `@dataclass` to package related state for `AttentionGroup`. Typical fields include `backend`, `layer_names`, `kv_cache_spec`, `kv_cache_group_id`, `metadata_builders`.
**CN:** `AttentionGroup` 使用 `@dataclass` 打包相关状态。典型字段包括 `backend`, `layer_names`, `kv_cache_spec`, `kv_cache_group_id`, `metadata_builders`。

### `AttentionGroup.create_metadata_builders` method / `AttentionGroup.create_metadata_builders` 方法
```python
    def create_metadata_builders(
        self,
        vllm_config,
        device,
        kernel_block_size: int | None = None,
        num_metadata_builders: int = 1,
    ):
        kv_cache_spec_builder = (
            self.kv_cache_spec.copy_with_new_block_size(kernel_block_size)
            if kernel_block_size is not None
            else self.kv_cache_spec
        )
        self.metadata_builders = [
            self.backend.get_builder_cls()(
                kv_cache_spec_builder,
                self.layer_names,
                vllm_config,
                device,
            )
            for _ in range(num_metadata_builders)
        ]
```
**EN:** This method creates a new object or plan within `AttentionGroup`. Key calls include `copy_with_new_block_size`, `get_builder_cls`, `range`. It touches state such as `metadata_builders`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会创建新的对象或计划，其作用域位于`AttentionGroup`。 关键调用包括 `copy_with_new_block_size`, `get_builder_cls`, `range`。 它会读写 `metadata_builders` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionGroup.get_metadata_builder` method / `AttentionGroup.get_metadata_builder` 方法
```python
    def get_metadata_builder(self, ubatch_id: int = 0) -> AttentionMetadataBuilder:
        assert len(self.metadata_builders) > ubatch_id
        return self.metadata_builders[ubatch_id]
```
**EN:** This method returns or derives a value within `AttentionGroup`. Key calls include `len`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionGroup`。 关键调用包括 `len`。

### `select_common_block_size` function / `select_common_block_size` 函数
```python
def select_common_block_size(
    kv_manager_block_size: int,
    backends: list[type[AttentionBackend]],
) -> int:
    """
    Select a block size that is supported by all backends and is a factor of
    kv_manager_block_size.

    If kv_manager_block_size is supported by all backends, return it directly.
    Otherwise, return the max supported size.

    Args:
        kv_manager_block_size: Block size of KV cache.
        backends: List of attention backend classes.

    Returns:
        The selected block size.

    Raises:
        ValueError: If no valid block size found.
    """

    def block_size_is_supported(
        backends: list[type[AttentionBackend]], block_size: int
    ) -> bool:
        """Check if the block size is supported by all backends."""
        for backend in backends:
            is_supported = False
            for supported_size in backend.get_supported_kernel_block_sizes():
                if isinstance(supported_size, int):
                    if block_size == supported_size:
                        is_supported = True
                elif isinstance(supported_size, MultipleOf):
                    if block_size % supported_size.base == 0:
                        is_supported = True
                else:
                    raise ValueError(f"Unknown supported size: {supported_size}")
            if not is_supported:
                return False
        return True

    # Case 1: if the block_size of kv cache manager is supported by all backends,
    # return it directly.
    if block_size_is_supported(backends, kv_manager_block_size):
        return kv_manager_block_size

    # Case 2: otherwise, the block_size must be an `int`-format supported size of
    # at least one backend. Iterate over all `int`-format supported sizes in
    # descending order and return the first one that is supported by all backends.
    # Simple proof:
    # If the supported size b is in MultipleOf(x_i) format for all attention
    # backends i, and b a factor of kv_manager_block_size, then
    # kv_manager_block_size also satisfies MultipleOf(x_i) for all i. We will
    # return kv_manager_block_size in case 1.
    all_int_supported_sizes = set(
        supported_size
        for backend in backends
        for supported_size in backend.get_supported_kernel_block_sizes()
        if isinstance(supported_size, int)
    )

    for supported_size in sorted(all_int_supported_sizes, reverse=True):
        if kv_manager_block_size % supported_size != 0:
            continue
        if block_size_is_supported(backends, supported_size):
            return supported_size
    raise ValueError(f"No common block size for {kv_manager_block_size}. ")
```
**EN:** This function implements `select_common_block_size` within the module. The docstring frames it as: Select a block size that is supported by all backends and is a factor of kv_manager_block_size. Key calls include `block_size_is_supported`, `set`, `sorted`, `ValueError`, `get_supported_kernel_block_sizes`, `isinstance`. The control flow contains 8 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `select_common_block_size`，其作用域位于the module。 关键调用包括 `block_size_is_supported`, `set`, `sorted`, `ValueError`, `get_supported_kernel_block_sizes`, `isinstance`。 控制流包含 8 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `prepare_kernel_block_sizes` function / `prepare_kernel_block_sizes` 函数
```python
def prepare_kernel_block_sizes(
    kv_cache_config: KVCacheConfig, attn_groups: list[list[AttentionGroup]]
) -> list[int]:
    """
    Generate kernel_block_sizes that matches each block_size.

    For attention backends that support virtual block splitting,
    use the supported block sizes from the backend.
    For other backends (like Mamba), use the same block size (no splitting).

    Args:
        kv_cache_config: The KV cache configuration.
        attn_groups: Attention groups indexed by KV cache group id.

    Returns:
        List of kernel block sizes for each cache group.
    """
    kernel_block_sizes = []
    for kv_cache_gid, kv_cache_group in enumerate(kv_cache_config.kv_cache_groups):
        kv_cache_spec = kv_cache_group.kv_cache_spec
        if isinstance(kv_cache_spec, UniformTypeKVCacheSpecs):
            # All layers in the UniformTypeKVCacheSpecs have the same type,
            # pick an arbitrary one to dispatch.
            kv_cache_spec = next(iter(kv_cache_spec.kv_cache_specs.values()))
        if isinstance(kv_cache_spec, EncoderOnlyAttentionSpec):
            continue
        if isinstance(kv_cache_spec, AttentionSpec):
            # This is an attention backend that supports virtual block splitting.
            kv_manager_block_size = kv_cache_group.kv_cache_spec.block_size
            group_backends = [g.backend for g in attn_groups[kv_cache_gid]]
            selected_kernel_size = select_common_block_size(
                kv_manager_block_size, group_backends
            )
            kernel_block_sizes.append(selected_kernel_size)
        elif isinstance(kv_cache_spec, MambaSpec):
            # This is likely Mamba or other non-attention cache, no splitting.
            kernel_block_sizes.append(kv_cache_spec.block_size)
        else:
            raise NotImplementedError(
                f"unknown kv cache spec {kv_cache_group.kv_cache_spec}"
            )
    return kernel_block_sizes
```
**EN:** This function prepares inputs and state within the module. The docstring frames it as: Generate kernel_block_sizes that matches each block_size. Key calls include `enumerate`, `isinstance`, `next`, `select_common_block_size`, `append`, `iter`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会准备输入与状态，其作用域位于the module。 关键调用包括 `enumerate`, `isinstance`, `next`, `select_common_block_size`, `append`, `iter`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `sanity_check_mm_encoder_outputs` function / `sanity_check_mm_encoder_outputs` 函数
```python
def sanity_check_mm_encoder_outputs(
    mm_embeddings: MultiModalEmbeddings,
    expected_num_items: int,
) -> None:
    """
    Perform sanity checks for the result of
    [`vllm.model_executor.models.SupportsMultiModal.embed_multimodal`][].
    """
    assert isinstance(mm_embeddings, (list, tuple, torch.Tensor)), (
        "Expected multimodal embeddings to be a list/tuple of 2D tensors, "
        f"or a single 3D tensor, but got {type(mm_embeddings)} "
        "instead. This is most likely due to incorrect implementation "
        "of the model's `embed_multimodal` method."
    )

    assert len(mm_embeddings) == expected_num_items, (
        "Expected number of multimodal embeddings to match number of "
        f"input items: {expected_num_items}, but got {len(mm_embeddings)=} "
        "instead. This is most likely due to incorrect implementation "
        "of the model's `embed_multimodal` method."
    )

    assert all(e.ndim == 2 for e in mm_embeddings), (
        "Expected multimodal embeddings to be a sequence of 2D tensors, "
        f"but got tensors with shapes {[e.shape for e in mm_embeddings]} "
        "instead. This is most likely due to incorrect implementation "
        "of the model's `embed_multimodal` method."
    )
```
**EN:** This function implements `sanity_check_mm_encoder_outputs` within the module. The docstring frames it as: Perform sanity checks for the result of [`vllm.model_executor.models.SupportsMultiModal.embed_multimodal`][]. Key calls include `isinstance`, `all`, `len`, `type`.
**CN:** 该函数会实现 `sanity_check_mm_encoder_outputs`，其作用域位于the module。 关键调用包括 `isinstance`, `all`, `len`, `type`。

### `request_memory` function / `request_memory` 函数
```python
def request_memory(init_snapshot: MemorySnapshot, cache_config: CacheConfig) -> int:
    """
    Calculate the amount of memory required by vLLM, then validate
    that the current amount of free memory is sufficient for that.
    """
    requested_memory = math.ceil(
        init_snapshot.total_memory * cache_config.gpu_memory_utilization
    )

    if init_snapshot.free_memory < requested_memory:
        raise ValueError(
            f"Free memory on device {init_snapshot.device_} "
            f"({format_gib(init_snapshot.free_memory)}/"
            f"{format_gib(init_snapshot.total_memory)} GiB) on startup "
            f"is less than desired GPU memory utilization "
            f"({cache_config.gpu_memory_utilization}, "
            f"{format_gib(requested_memory)} GiB). Decrease GPU memory "
            f"utilization or reduce GPU memory used by other processes."
        )

    return requested_memory
```
**EN:** This function implements `request_memory` within the module. The docstring frames it as: Calculate the amount of memory required by vLLM, then validate that the current amount of free memory is sufficient for that. Key calls include `ceil`, `ValueError`, `format_gib`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `request_memory`，其作用域位于the module。 关键调用包括 `ceil`, `ValueError`, `format_gib`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `add_kv_sharing_layers_to_kv_cache_groups` function / `add_kv_sharing_layers_to_kv_cache_groups` 函数
```python
def add_kv_sharing_layers_to_kv_cache_groups(
    shared_kv_cache_layers: dict[str, str],
    kv_cache_groups: list[KVCacheGroupSpec],
    runner_only_attn_layers: set[str] | None = None,
) -> None:
    """
    Sets up KV cache sharing by reusing the allocated KV caches in `kv_caches`
    for layers that do not allocate its own KV cache, based on the mapping in
    `shared_kv_cache_layers`. Adds these layers to the corresponding KV cache
    group, which is needed to ensure that attention metadata is assigned later.

    Args:
        shared_kv_cache_layers: Layer pairings for cross-layer KV sharing.
            If an Attention layer `layer_name` is in the keys of this dict, it
            means this layer will perform attention using the keys and values
            from the KV cache of `shared_kv_cache_layers[layer_name]`.
        kv_cache_groups: The KV cache groups of the model.
    """
    layer_to_kv_cache_group: dict[str, KVCacheGroupSpec] = {}
    for kv_cache_group in kv_cache_groups:
        for layer_name in kv_cache_group.layer_names:
            layer_to_kv_cache_group[layer_name] = kv_cache_group

    for layer_name, target_layer_name in shared_kv_cache_layers.items():
        tgt_kv_cache_group = layer_to_kv_cache_group[target_layer_name]
        tgt_kv_cache_group.layer_names.append(layer_name)

        if runner_only_attn_layers is not None:
            runner_only_attn_layers.add(layer_name)
```
**EN:** This function implements `add_kv_sharing_layers_to_kv_cache_groups` within the module. The docstring frames it as: Sets up KV cache sharing by reusing the allocated KV caches in `kv_caches` for layers that do not allocate its own KV cache, based on the mapping in `shared_kv_cache_layers`. Key calls include `items`, `append`, `add`. The control flow contains 1 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `add_kv_sharing_layers_to_kv_cache_groups`，其作用域位于the module。 关键调用包括 `items`, `append`, `add`。 控制流包含 1 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `bind_kv_cache` function / `bind_kv_cache` 函数
```python
def bind_kv_cache(
    kv_caches: dict[str, torch.Tensor],
    forward_context: dict[str, Attention],
    runner_kv_caches: list[torch.Tensor],
    num_attn_module: int = 1,
) -> None:
    """
    Bind the allocated KV cache to both ModelRunner and forward context so
    that the KV cache can be used in the forward pass.

    This function:
      1) Fills the ModelRunner's kv cache list (`runner_kv_caches`) with
         kv_caches.
      2) Associates each attention layer in the `forward_context` with its
         corresponding KV cache in kv_caches.

    Args:
        kv_caches: The allocated kv_caches with layer names as keys.
        forward_context: The global forward context containing all Attention
            layers with layer names as keys.
        runner_kv_caches: The kv_cache declared by ModelRunner.
    """
    # Bind kv_caches to ModelRunner
    assert len(runner_kv_caches) == 0

    # Convert kv_caches dict to a list of tensors in the order of layer_index.
    index2name = defaultdict(list)
    for layer_name in kv_caches:
        index2name[extract_layer_index(layer_name, num_attn_module)].append(layer_name)

    for layer_index in sorted(index2name.keys()):
        layer_names = index2name[layer_index]
        if len(layer_names) > 1:
            # One typical case is encoder-decoder model, e.g., bart.
            # The cross attention and self attention in the same decoder layer
            # has different layer_name but the same layer_index.

            # TODO - analyze where runner_kv_caches is used and the right
            # way to ensure it properly reflects multiple attention layers
            # in the same decoder block.
            if (
                current_platform.is_cuda_alike()
                or current_platform.is_xpu()
                or current_platform.is_cpu()
            ):
                # We know that the GPU / CPU runner is not impacted by this
                # case. Some test code depends on runner_kv_caches, but
                # not in a way that's impacted by ignoring this.
                pass
            else:
                raise NotImplementedError
        for layer_name in layer_names:
            runner_kv_caches.append(kv_caches[layer_name])

    # Bind kv_caches to forward context
    for layer_name, kv_cache in kv_caches.items():
        forward_context[layer_name].kv_cache = kv_cache
```
**EN:** This function implements `bind_kv_cache` within the module. The docstring frames it as: Bind the allocated KV cache to both ModelRunner and forward context so that the KV cache can be used in the forward pass. Key calls include `defaultdict`, `sorted`, `items`, `len`, `append`, `keys`. The control flow contains 2 branch(es) and 4 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `bind_kv_cache`，其作用域位于the module。 关键调用包括 `defaultdict`, `sorted`, `items`, `len`, `append`, `keys`。 控制流包含 2 个分支和 4 个循环，说明这里承担了较强的协调逻辑。

### `is_residual_scattered_for_sp` function / `is_residual_scattered_for_sp` 函数
```python
def is_residual_scattered_for_sp(
    vllm_config: VllmConfig, num_input_tokens: int
) -> bool:
    """Check if the residual tensor is scattered for sequence parallelism.

    The residual tensor is scattered across tensor parallel ranks when sequence
    parallelism and tensor parallelism is enabled. SP is only supported in
    full-graph compilation mode.
    """
    if not vllm_config.compilation_config.pass_config.enable_sp:
        return False

    tp = vllm_config.parallel_config.tensor_parallel_size

    if tp == 1:
        return False

    assert (
        vllm_config.compilation_config.use_inductor_graph_partition
        or not vllm_config.compilation_config.splitting_ops
    ), "Sequence parallelism requires full-graph compilation"

    # When sequence parallelism is enabled, we always pad num_input_tokens
    # to be a multiple of tensor_parallel_size (tp) earlier.
    assert num_input_tokens % tp == 0

    return True
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Check if the residual tensor is scattered for sequence parallelism. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_zero_kv_blocks_kernel`: top-level helper or orchestration entry point. / `_zero_kv_blocks_kernel`：顶层辅助函数或编排入口。
- `KVBlockZeroer`: central class or interface in this module. / `KVBlockZeroer`：本模块中的核心类或接口。
- `AttentionGroup`: central class or interface in this module. / `AttentionGroup`：本模块中的核心类或接口。
- `select_common_block_size`: top-level helper or orchestration entry point. / `select_common_block_size`：顶层辅助函数或编排入口。
- `prepare_kernel_block_sizes`: top-level helper or orchestration entry point. / `prepare_kernel_block_sizes`：顶层辅助函数或编排入口。
- `sanity_check_mm_encoder_outputs`: top-level helper or orchestration entry point. / `sanity_check_mm_encoder_outputs`：顶层辅助函数或编排入口。
- `request_memory`: top-level helper or orchestration entry point. / `request_memory`：顶层辅助函数或编排入口。
- `add_kv_sharing_layers_to_kv_cache_groups`: top-level helper or orchestration entry point. / `add_kv_sharing_layers_to_kv_cache_groups`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `math`, `collections`, `dataclasses`, `itertools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.model_executor.layers.attention`, `vllm.model_executor.models.interfaces`, `vllm.model_executor.models.utils`, `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.mem_utils`, `vllm.v1.attention.backend`, `vllm.v1.kv_cache_interface`
