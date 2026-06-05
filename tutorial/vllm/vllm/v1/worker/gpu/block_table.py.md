# block_table.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/block_table.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `BlockTables`, `_gather_block_tables_kernel`, `_compute_slot_mappings_kernel` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `BlockTables`, `_gather_block_tables_kernel`, `_compute_slot_mappings_kernel`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Iterable

import torch

from vllm.triton_utils import tl, triton
from vllm.v1.attention.backends.utils import PAD_SLOT_ID
from vllm.v1.worker.gpu.buffer_utils import StagedWriteTensor, UvaBackedTensor
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.triton_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.worker.gpu.buffer_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.triton_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.worker.gpu.buffer_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `BlockTables` class / `BlockTables` 类
```python
class BlockTables:
```
**EN:** Introduces the `BlockTables` class. Core methods include `__init__`, `_make_ptr_tensor`, `init_block_table_layout_tensors`, `append_block_ids`, `apply_staged_writes`, `gather_block_tables`.
**CN:** 这里定义 `BlockTables` 类。核心方法包括 `__init__`, `_make_ptr_tensor`, `init_block_table_layout_tensors`, `append_block_ids`, `apply_staged_writes`, `gather_block_tables`。

### `BlockTables.__init__` method / `BlockTables.__init__` 方法
```python
    def __init__(
        self,
        block_sizes: list[int],
        max_num_reqs: int,
        max_num_batched_tokens: int,
        max_num_blocks_per_group: list[int],
        device: torch.device,
        cp_size: int = 1,
        cp_rank: int = 0,
        cp_interleave: int = 1,
    ):
        self.block_sizes = block_sizes
        self.max_num_reqs = max_num_reqs
        self.max_num_batched_tokens = max_num_batched_tokens
        self.device = device

        self.cp_size = cp_size
        self.cp_rank = cp_rank
        self.cp_interleave = cp_interleave

        self.num_kv_cache_groups = len(self.block_sizes)
        assert len(max_num_blocks_per_group) == self.num_kv_cache_groups
        # num_kv_cache_groups x [max_num_reqs, max_num_blocks]
        self.block_tables: list[StagedWriteTensor] = []
        for i in range(self.num_kv_cache_groups):
            max_num_blocks = max_num_blocks_per_group[i]
            block_table = StagedWriteTensor(
                (self.max_num_reqs, max_num_blocks),
                dtype=torch.int32,
                device=device,
            )
            self.block_tables.append(block_table)

        self.num_blocks = UvaBackedTensor(
            (self.num_kv_cache_groups, self.max_num_reqs),
            dtype=torch.int32,
        )

        # Block tables used for model's forward pass.
        # num_kv_cache_groups x [max_num_reqs, max_num_blocks]
        self.input_block_tables: list[torch.Tensor] = [
            torch.zeros_like(b.gpu) for b in self.block_tables
        ]

        self.slot_mappings = torch.zeros(
            self.num_kv_cache_groups,
            self.max_num_batched_tokens,
            dtype=torch.int64,
            device=self.device,
        )

        self.init_block_table_layout_tensors()
```
**EN:** This method initializes the object state within `BlockTables`. Key calls include `len`, `range`, `UvaBackedTensor`, `zeros`, `init_block_table_layout_tensors`, `StagedWriteTensor`. It touches state such as `block_sizes`, `max_num_reqs`, `max_num_batched_tokens`, `device`, `cp_size`, `cp_rank`, `cp_interleave`, `num_kv_cache_groups`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockTables`。 关键调用包括 `len`, `range`, `UvaBackedTensor`, `zeros`, `init_block_table_layout_tensors`, `StagedWriteTensor`。 它会读写 `block_sizes`, `max_num_reqs`, `max_num_batched_tokens`, `device`, `cp_size`, `cp_rank`, `cp_interleave`, `num_kv_cache_groups` 等状态。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BlockTables.init_block_table_layout_tensors` method / `BlockTables.init_block_table_layout_tensors` 方法
```python
    def init_block_table_layout_tensors(self) -> None:
        # Called at init and after a CuMem kv_cache wake-up. The ptr tensors
        # cache raw data_ptr() values that go stale once the underlying tensors
        # are reallocated on wake; block_sizes_tensor needs re-populating
        # because its storage lives under the kv_cache pool tag and comes back
        # with undefined contents.
        self.block_table_ptrs = self._make_ptr_tensor(
            [b.gpu for b in self.block_tables]
        )
        self.block_table_strides = torch.tensor(
            [b.gpu.stride(0) for b in self.block_tables],
            dtype=torch.int64,
            device=self.device,
        )
        self.block_sizes_tensor = torch.tensor(
            self.block_sizes, dtype=torch.int32, device=self.device
        )
        self.input_block_table_ptrs = self._make_ptr_tensor(self.input_block_tables)
```
**EN:** This method implements `init_block_table_layout_tensors` within `BlockTables`. Key calls include `_make_ptr_tensor`, `tensor`, `stride`. It touches state such as `block_table_ptrs`, `block_table_strides`, `block_sizes_tensor`, `input_block_table_ptrs`.
**CN:** 该方法会实现 `init_block_table_layout_tensors`，其作用域位于`BlockTables`。 关键调用包括 `_make_ptr_tensor`, `tensor`, `stride`。 它会读写 `block_table_ptrs`, `block_table_strides`, `block_sizes_tensor`, `input_block_table_ptrs` 等状态。

### `BlockTables.append_block_ids` method / `BlockTables.append_block_ids` 方法
```python
    def append_block_ids(
        self,
        req_index: int,
        new_block_ids: tuple[list[int], ...],
        overwrite: bool,
    ) -> None:
        for i in range(self.num_kv_cache_groups):
            start = self.num_blocks.np[i, req_index] if not overwrite else 0
            block_ids = new_block_ids[i]
            self.block_tables[i].stage_write(req_index, start, block_ids)
            self.num_blocks.np[i, req_index] = start + len(block_ids)
```
**EN:** This method implements `append_block_ids` within `BlockTables`. Key calls include `range`, `stage_write`, `len`. It touches state such as `num_blocks`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `append_block_ids`，其作用域位于`BlockTables`。 关键调用包括 `range`, `stage_write`, `len`。 它会读写 `num_blocks` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BlockTables.apply_staged_writes` method / `BlockTables.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        # TODO(woosuk): This can be inefficient since it launches one kernel per
        # block table. Implement a kernel to handle all block tables at once.
        for block_table in self.block_tables:
            block_table.apply_write()
        self.num_blocks.copy_to_uva()
```
**EN:** This method implements `apply_staged_writes` within `BlockTables`. Key calls include `copy_to_uva`, `apply_write`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`BlockTables`。 关键调用包括 `copy_to_uva`, `apply_write`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BlockTables.gather_block_tables` method / `BlockTables.gather_block_tables` 方法
```python
    def gather_block_tables(
        self,
        idx_mapping: torch.Tensor,
        num_reqs_padded: int,
    ) -> tuple[torch.Tensor, ...]:
        num_reqs = idx_mapping.shape[0]
        # Launch kernel with num_reqs_padded to fuse zeroing of padded rows.
        _gather_block_tables_kernel[(self.num_kv_cache_groups, num_reqs_padded)](
            idx_mapping,
            self.block_table_ptrs,
            self.input_block_table_ptrs,
            self.block_table_strides,
            self.num_blocks.gpu,
            self.num_blocks.gpu.stride(0),
            num_reqs,
            self.input_block_tables[0].shape[1],  # max_num_blocks
            BLOCK_SIZE=1024,  # type: ignore
        )
        return tuple(bt[:num_reqs_padded] for bt in self.input_block_tables)
```
**EN:** This method implements `gather_block_tables` within `BlockTables`. Key calls include `tuple`, `stride`.
**CN:** 该方法会实现 `gather_block_tables`，其作用域位于`BlockTables`。 关键调用包括 `tuple`, `stride`。

### `BlockTables.get_dummy_block_tables` method / `BlockTables.get_dummy_block_tables` 方法
```python
    def get_dummy_block_tables(self, num_reqs: int) -> tuple[torch.Tensor, ...]:
        # NOTE(woosuk): The output may be used for CUDA graph capture.
        # Therefore, this method must return the persistent tensor
        # with the same memory address as that used during the model's forward pass,
        # rather than allocating a new tensor.
        return tuple(block_table[:num_reqs] for block_table in self.input_block_tables)
```
**EN:** This method returns or derives a value within `BlockTables`. Key calls include `tuple`.
**CN:** 该方法会返回或推导一个值，其作用域位于`BlockTables`。 关键调用包括 `tuple`。

### `BlockTables.compute_slot_mappings` method / `BlockTables.compute_slot_mappings` 方法
```python
    def compute_slot_mappings(
        self,
        idx_mapping: torch.Tensor,
        query_start_loc: torch.Tensor,
        positions: torch.Tensor,
        num_tokens_padded: int,
    ) -> torch.Tensor:
        num_reqs = idx_mapping.shape[0]
        num_groups = self.num_kv_cache_groups
        _compute_slot_mappings_kernel[(num_groups, num_reqs + 1)](
            self.max_num_batched_tokens,
            idx_mapping,
            query_start_loc,
            positions,
            self.block_table_ptrs,
            self.block_table_strides,
            self.block_sizes_tensor,
            self.slot_mappings,
            self.slot_mappings.stride(0),
            self.cp_rank,
            CP_SIZE=self.cp_size,
            CP_INTERLEAVE=self.cp_interleave,
            PAD_ID=PAD_SLOT_ID,
            TRITON_BLOCK_SIZE=1024,  # type: ignore
        )
        return self.slot_mappings[:, :num_tokens_padded]
```
**EN:** This method computes derived values within `BlockTables`. Key calls include `stride`.
**CN:** 该方法会计算派生值，其作用域位于`BlockTables`。 关键调用包括 `stride`。

### `BlockTables.get_dummy_slot_mappings` method / `BlockTables.get_dummy_slot_mappings` 方法
```python
    def get_dummy_slot_mappings(self, num_tokens: int) -> torch.Tensor:
        # Fill the entire slot_mappings tensor, not just the first `num_tokens` entries.
        # This is because the padding logic is complex and kernels may access beyond
        # the requested range.
        self.slot_mappings.fill_(PAD_SLOT_ID)
        # NOTE(woosuk): The output may be used for CUDA graph capture.
        # Therefore, this method must return the persistent tensor
        # with the same memory address as that used during the model's forward pass,
        # rather than allocating a new tensor.
        return self.slot_mappings[:, :num_tokens]
```
**EN:** This method returns or derives a value within `BlockTables`. Key calls include `fill_`.
**CN:** 该方法会返回或推导一个值，其作用域位于`BlockTables`。 关键调用包括 `fill_`。

### `_gather_block_tables_kernel` function / `_gather_block_tables_kernel` 函数
```python
@triton.jit(do_not_specialize=["num_reqs"])
def _gather_block_tables_kernel(
    batch_idx_to_req_idx,  # [batch_size]
    src_block_table_ptrs,  # [num_kv_cache_groups]
    dst_block_table_ptrs,  # [num_kv_cache_groups]
    block_table_strides,  # [num_kv_cache_groups]
    num_blocks_ptr,  # [num_kv_cache_groups, max_num_reqs]
    num_blocks_stride,
    num_reqs,  # actual number of requests (for padding)
    max_num_blocks,  # stride for zeroing padded rows
    BLOCK_SIZE: tl.constexpr,
):
    # kv cache group id
    group_id = tl.program_id(0)
    batch_idx = tl.program_id(1)

    stride = tl.load(block_table_strides + group_id)
    dst_block_table_ptr = _load_ptr(dst_block_table_ptrs + group_id, tl.int32)
    dst_row_ptr = dst_block_table_ptr + batch_idx * stride

    if batch_idx >= num_reqs:
        # Zero out padded rows.
        for i in tl.range(0, max_num_blocks, BLOCK_SIZE):
            offset = i + tl.arange(0, BLOCK_SIZE)
            tl.store(dst_row_ptr + offset, 0, mask=offset < max_num_blocks)
        return

    req_idx = tl.load(batch_idx_to_req_idx + batch_idx)
    group_num_blocks_ptr = num_blocks_ptr + group_id * num_blocks_stride
    num_blocks = tl.load(group_num_blocks_ptr + req_idx)

    src_block_table_ptr = _load_ptr(src_block_table_ptrs + group_id, tl.int32)
    src_row_ptr = src_block_table_ptr + req_idx * stride

    for i in tl.range(0, num_blocks, BLOCK_SIZE):
        offset = i + tl.arange(0, BLOCK_SIZE)
        block_ids = tl.load(src_row_ptr + offset, mask=offset < num_blocks)
        tl.store(dst_row_ptr + offset, block_ids, mask=offset < num_blocks)
```
**EN:** This function implements `_gather_block_tables_kernel` within the module. Key calls include `jit`, `program_id`, `load`, `_load_ptr`, `range`, `store`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_gather_block_tables_kernel`，其作用域位于the module。 关键调用包括 `jit`, `program_id`, `load`, `_load_ptr`, `range`, `store`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `_compute_slot_mappings_kernel` function / `_compute_slot_mappings_kernel` 函数
```python
@triton.jit
def _compute_slot_mappings_kernel(
    max_num_tokens,
    idx_mapping,  # [num_reqs]
    query_start_loc,  # [num_reqs + 1]
    pos,  # [num_tokens]
    block_table_ptrs,  # [num_kv_cache_groups]
    block_table_strides,  # [num_kv_cache_groups]
    block_sizes,  # [num_kv_cache_groups]
    slot_mappings_ptr,  # [num_kv_cache_groups, max_num_tokens]
    slot_mappings_stride,
    cp_rank,
    CP_SIZE: tl.constexpr,
    CP_INTERLEAVE: tl.constexpr,
    PAD_ID: tl.constexpr,
    TRITON_BLOCK_SIZE: tl.constexpr,
):
    # kv cache group id
    group_id = tl.program_id(0)
    batch_idx = tl.program_id(1)
    slot_mapping_ptr = slot_mappings_ptr + group_id * slot_mappings_stride

    if batch_idx == tl.num_programs(1) - 1:
        # Pad remaining slots to -1. This is needed for CUDA graphs.
        # Start from actual token count (not padded) to cover the gap
        # between actual tokens and padded tokens that can contain stale
        # valid slot IDs from previous chunks during chunked prefill.
        actual_num_tokens = tl.load(query_start_loc + batch_idx)
        for i in range(actual_num_tokens, max_num_tokens, TRITON_BLOCK_SIZE):
            offset = i + tl.arange(0, TRITON_BLOCK_SIZE)
            tl.store(slot_mapping_ptr + offset, PAD_ID, mask=offset < max_num_tokens)
        return

    block_table_ptr = _load_ptr(block_table_ptrs + group_id, tl.int32)
    block_table_stride = tl.load(block_table_strides + group_id)
    block_size = tl.load(block_sizes + group_id)

    req_state_idx = tl.load(idx_mapping + batch_idx)
    start_idx = tl.load(query_start_loc + batch_idx)
    end_idx = tl.load(query_start_loc + batch_idx + 1)
    for i in range(start_idx, end_idx, TRITON_BLOCK_SIZE):
        offset = i + tl.arange(0, TRITON_BLOCK_SIZE)
        positions = tl.load(pos + offset, mask=offset < end_idx, other=0)

        block_indices = positions // (block_size * CP_SIZE)
        block_offsets = positions % (block_size * CP_SIZE)
        block_numbers = tl.load(
            block_table_ptr + req_state_idx * block_table_stride + block_indices
        )

        if CP_SIZE == 1:
            # Common case: Context parallelism is not used.
            slot_ids = block_numbers * block_size + block_offsets
        else:
            # Context parallelism is used.
            is_local = block_offsets // CP_INTERLEAVE % CP_SIZE == cp_rank
            rounds = block_offsets // (CP_INTERLEAVE * CP_SIZE)
            remainder = block_offsets % CP_INTERLEAVE
            local_offsets = rounds * CP_INTERLEAVE + remainder
            slot_ids = block_numbers * block_size + local_offsets
            slot_ids = tl.where(is_local, slot_ids, PAD_ID)

        tl.store(slot_mapping_ptr + offset, slot_ids, mask=offset < end_idx)
```
**EN:** This function implements `_compute_slot_mappings_kernel` within the module. Key calls include `program_id`, `_load_ptr`, `load`, `range`, `store`, `num_programs`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_compute_slot_mappings_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `_load_ptr`, `load`, `range`, `store`, `num_programs`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `_load_ptr` function / `_load_ptr` 函数
```python
@triton.jit
def _load_ptr(ptr_to_ptr, elem_dtype):
    ptr = tl.load(ptr_to_ptr)
    ptr = tl.cast(ptr, tl.pointer_type(elem_dtype))
    return tl.multiple_of(ptr, 16)
```
**EN:** This function implements `_load_ptr` within the module. Key calls include `load`, `cast`, `multiple_of`, `pointer_type`.
**CN:** 该函数会实现 `_load_ptr`，其作用域位于the module。 关键调用包括 `load`, `cast`, `multiple_of`, `pointer_type`。

## Key Concepts / 关键概念
- `BlockTables`: central class or interface in this module. / `BlockTables`：本模块中的核心类或接口。
- `_gather_block_tables_kernel`: top-level helper or orchestration entry point. / `_gather_block_tables_kernel`：顶层辅助函数或编排入口。
- `_compute_slot_mappings_kernel`: top-level helper or orchestration entry point. / `_compute_slot_mappings_kernel`：顶层辅助函数或编排入口。
- `_load_ptr`: top-level helper or orchestration entry point. / `_load_ptr`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.triton_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.worker.gpu.buffer_utils`
