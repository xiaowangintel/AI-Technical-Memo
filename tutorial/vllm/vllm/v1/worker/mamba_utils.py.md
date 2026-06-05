# mamba_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/mamba_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `batch_memcpy_kernel`, `batch_memcpy`, `get_mamba_groups` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `batch_memcpy_kernel`, `batch_memcpy`, `get_mamba_groups`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import dataclasses
import itertools
from collections.abc import Callable
from typing import Any

import torch

from vllm.config import CacheConfig
from vllm.model_executor.layers.mamba.mamba_utils import (
    MambaStateCopyFunc,
)
from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.kv_cache_interface import KVCacheConfig, MambaSpec
from vllm.v1.utils import CpuGpuBuffer
from vllm.v1.worker.gpu_input_batch import CachedRequestState
from vllm.v1.worker.lora_model_runner_mixin import GPUInputBatch
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.triton_utils`, `vllm.utils.math_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.triton_utils`, `vllm.utils.math_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `batch_memcpy_kernel` function / `batch_memcpy_kernel` 函数
```python
@triton.jit
def batch_memcpy_kernel(src_ptrs, dst_ptrs, sizes, BLOCK_SIZE: tl.constexpr):
    pid = tl.program_id(0)

    src_ptr = tl.load(src_ptrs + pid)
    dst_ptr = tl.load(dst_ptrs + pid)
    size = tl.load(sizes + pid)

    offsets = tl.arange(0, BLOCK_SIZE)
    for i in range(0, size, BLOCK_SIZE):
        mask = (i + offsets) < size

        curr_src_ptr = (src_ptr + i + offsets).to(tl.pointer_type(tl.uint8))
        curr_dst_ptr = (dst_ptr + i + offsets).to(tl.pointer_type(tl.uint8))

        data = tl.load(curr_src_ptr, mask=mask)
        tl.store(curr_dst_ptr, data, mask=mask)
```
**EN:** This function implements `batch_memcpy_kernel` within the module. Key calls include `program_id`, `load`, `arange`, `range`, `to`, `store`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `batch_memcpy_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `range`, `to`, `store`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `batch_memcpy` function / `batch_memcpy` 函数
```python
def batch_memcpy(src_ptrs, dst_ptrs, sizes):
    batch = src_ptrs.shape[0]
    assert dst_ptrs.shape[0] == batch
    assert sizes.shape[0] == batch

    grid = (batch,)
    BLOCK_SIZE = 1024
    batch_memcpy_kernel[grid](src_ptrs, dst_ptrs, sizes, BLOCK_SIZE=BLOCK_SIZE)
```
**EN:** This function implements `batch_memcpy` within the module.
**CN:** 该函数会实现 `batch_memcpy`，其作用域位于the module。

### `get_mamba_groups` function / `get_mamba_groups` 函数
```python
def get_mamba_groups(kv_cache_config: KVCacheConfig) -> tuple[list[int], MambaSpec]:
    mamba_group_ids: list[int] = []
    mamba_specs: list[MambaSpec] = []
    for i in range(len(kv_cache_config.kv_cache_groups)):
        kv_cache_spec = kv_cache_config.kv_cache_groups[i].kv_cache_spec
        if isinstance(kv_cache_spec, MambaSpec):
            mamba_group_ids.append(i)
            mamba_specs.append(kv_cache_spec)
    assert len(mamba_group_ids) > 0, "no mamba layers in the model"
    assert all(mamba_specs[0] == spec for spec in mamba_specs)
    return mamba_group_ids, mamba_specs[0]
```
**EN:** This function returns or derives a value within the module. Key calls include `range`, `all`, `len`, `isinstance`, `append`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `range`, `all`, `len`, `isinstance`, `append`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `MambaCopyBuffers` class / `MambaCopyBuffers` 类
```python
@dataclasses.dataclass
class MambaCopyBuffers:
    src_ptrs: CpuGpuBuffer
    dst_ptrs: CpuGpuBuffer
    sizes: CpuGpuBuffer
    mamba_group_ids: list[int]
    mamba_spec: MambaSpec
    offset: int = 0
```
**EN:** Introduces the `MambaCopyBuffers` class. Core methods include `create`.
**CN:** 这里定义 `MambaCopyBuffers` 类。核心方法包括 `create`。

### `MambaCopyBuffers.create` method / `MambaCopyBuffers.create` 方法
```python
    @classmethod
    def create(
        cls,
        max_num_reqs: int,
        kv_cache_config: KVCacheConfig,
        copy_funcs: tuple[MambaStateCopyFunc, ...],
        make_buffer: Callable[..., CpuGpuBuffer],
    ) -> "MambaCopyBuffers":
        mamba_group_ids, mamba_spec = get_mamba_groups(kv_cache_config)
        entries_per_req = sum(
            len(kv_cache_config.kv_cache_groups[gid].layer_names)
            for gid in mamba_group_ids
        ) * len(copy_funcs)
        n = max_num_reqs * entries_per_req
        return cls(
            src_ptrs=make_buffer(n, dtype=torch.int64),
            dst_ptrs=make_buffer(n, dtype=torch.int64),
            sizes=make_buffer(n, dtype=torch.int32),
            mamba_group_ids=mamba_group_ids,
            mamba_spec=mamba_spec,
        )
```
**EN:** This method creates a new object or plan within `MambaCopyBuffers`. Key calls include `get_mamba_groups`, `cls`, `sum`, `len`, `make_buffer`.
**CN:** 该方法会创建新的对象或计划，其作用域位于`MambaCopyBuffers`。 关键调用包括 `get_mamba_groups`, `cls`, `sum`, `len`, `make_buffer`。

### `collect_mamba_copy_meta` function / `collect_mamba_copy_meta` 函数
```python
def collect_mamba_copy_meta(
    copy_bufs: MambaCopyBuffers,
    kv_cache_config: KVCacheConfig,
    mamba_state_copy_funcs: tuple[MambaStateCopyFunc, ...],
    mamba_group_ids: list[int],
    src_block_idx: int,
    dest_block_idx: int,
    accept_token_bias: int,
    req_state: CachedRequestState,
    forward_context: dict[str, Any],
) -> None:
    if src_block_idx == dest_block_idx and accept_token_bias == 0:
        return

    src_ptrs_np = copy_bufs.src_ptrs.np
    dst_ptrs_np = copy_bufs.dst_ptrs.np
    sizes_np = copy_bufs.sizes.np
    offset = copy_bufs.offset

    for mamba_group_id in mamba_group_ids:
        block_ids = req_state.block_ids[mamba_group_id]
        dest_block_id = block_ids[dest_block_idx]
        layer_names = kv_cache_config.kv_cache_groups[mamba_group_id].layer_names
        for layer_name in layer_names:
            attention = forward_context[layer_name]
            kv_caches: list[torch.Tensor] = attention.kv_cache
            for state, state_copy_func in zip(kv_caches, mamba_state_copy_funcs):
                copy_spec = state_copy_func(
                    state, block_ids, src_block_idx, accept_token_bias + 1
                )

                src_ptrs_np[offset] = copy_spec.start_addr
                dst_ptrs_np[offset] = state[dest_block_id].data_ptr()
                sizes_np[offset] = copy_spec.num_elements * state.element_size()
                offset += 1

    copy_bufs.offset = offset
```
**EN:** This function collects data from collaborators within the module. Key calls include `zip`, `state_copy_func`, `data_ptr`, `element_size`. The control flow contains 1 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会从协作组件收集数据，其作用域位于the module。 关键调用包括 `zip`, `state_copy_func`, `data_ptr`, `element_size`。 控制流包含 1 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `do_mamba_copy_block` function / `do_mamba_copy_block` 函数
```python
def do_mamba_copy_block(copy_bufs: MambaCopyBuffers):
    n = copy_bufs.offset
    if n == 0:
        return
    batch_memcpy(
        copy_bufs.src_ptrs.copy_to_gpu(n),
        copy_bufs.dst_ptrs.copy_to_gpu(n),
        copy_bufs.sizes.copy_to_gpu(n),
    )
```
**EN:** This function implements `do_mamba_copy_block` within the module. Key calls include `batch_memcpy`, `copy_to_gpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `do_mamba_copy_block`，其作用域位于the module。 关键调用包括 `batch_memcpy`, `copy_to_gpu`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `cleanup_mamba_state_idx` function / `cleanup_mamba_state_idx` 函数
```python
def cleanup_mamba_state_idx(
    scheduler_output: SchedulerOutput,
    mamba_state_idx: dict[str, int],
) -> None:
    """Pop stale `mamba_state_idx` entries for finished/preempted/resumed reqs.

    Force-preempted requests (e.g., during reset_prefix_cache / KV cache
    flush) appear in resumed_req_ids without a corresponding entry in
    preempted_req_ids, leaving stale entries that can point to block
    indices beyond the new (smaller) block allocation.
    """
    finished_req_ids = scheduler_output.finished_req_ids
    preempted_req_ids = scheduler_output.preempted_req_ids or set()
    resumed_req_ids = scheduler_output.scheduled_cached_reqs.resumed_req_ids
    for req_id in itertools.chain(finished_req_ids, preempted_req_ids, resumed_req_ids):
        mamba_state_idx.pop(req_id, None)
```
**EN:** This function implements `cleanup_mamba_state_idx` within the module. The docstring frames it as: Pop stale `mamba_state_idx` entries for finished/preempted/resumed reqs. Key calls include `chain`, `set`, `pop`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `cleanup_mamba_state_idx`，其作用域位于the module。 关键调用包括 `chain`, `set`, `pop`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `preprocess_mamba` function / `preprocess_mamba` 函数
```python
def preprocess_mamba(
    scheduler_output: SchedulerOutput,
    kv_cache_config: KVCacheConfig,
    cache_config: CacheConfig,
    mamba_state_idx: dict[str, int],
    input_batch: GPUInputBatch,
    requests: dict[str, CachedRequestState],
    forward_context: dict[str, Any],
    mamba_state_copy_funcs: tuple[MambaStateCopyFunc, ...],
    copy_bufs: MambaCopyBuffers,
):
    """
    Copy the mamba state of previous step to the last
    (1 + num_speculative_blocks) block.
    """
    mamba_group_ids = copy_bufs.mamba_group_ids
    mamba_spec = copy_bufs.mamba_spec
    num_speculative_blocks = mamba_spec.num_speculative_blocks
    # TODO(Chen): we need to optimize this function a lot
    assert cache_config.enable_prefix_caching
    block_size = mamba_spec.block_size
    cleanup_mamba_state_idx(scheduler_output, mamba_state_idx)

    copy_bufs.offset = 0
    for i, req_id in enumerate(input_batch.req_ids):
        req_state = requests[req_id]
        prev_state_idx = mamba_state_idx.get(req_id)
        if prev_state_idx is None:
            # new / resumed request, no previous state
            # if num_computed_tokens is 0, prev_state_idx will be -1
            prev_state_idx = (req_state.num_computed_tokens - 1) // block_size

        num_scheduled_tokens = scheduler_output.num_scheduled_tokens[req_id]
        num_blocks: int = (
            cdiv(req_state.num_computed_tokens + num_scheduled_tokens, block_size)
            + num_speculative_blocks
        )

        # We always save the current running state at the last
        # (1 + num_speculative_blocks) block.
        # A corner case worth mention here: assume we have block_size = 4 and
        # num_speculative_tokens = 2. The request is [A, B, C] and contains 2 draft
        # tokens [draft 1, draft 2]. Then we will have:
        # Block 0: [A, B, C, draft 1]
        # Block 1: [draft 2, TOFILL, TOFILL, TOFILL]
        # Block 2: speculative block
        # Block 3: speculative block
        # And use block 1 to save the running state.
        curr_state_idx = num_blocks - 1 - num_speculative_blocks
        mamba_state_idx[req_id] = curr_state_idx
        if prev_state_idx != -1 and prev_state_idx != curr_state_idx:
            collect_mamba_copy_meta(
                copy_bufs,
                kv_cache_config,
                mamba_state_copy_funcs,
                mamba_group_ids,
                prev_state_idx,
                curr_state_idx,
                input_batch.num_accepted_tokens_cpu[i] - 1,
                req_state,
                forward_context,
            )
            input_batch.num_accepted_tokens_cpu[i] = 1
    do_mamba_copy_block(copy_bufs)
```
**EN:** This function implements `preprocess_mamba` within the module. The docstring frames it as: Copy the mamba state of previous step to the last (1 + num_speculative_blocks) block. Key calls include `cleanup_mamba_state_idx`, `enumerate`, `do_mamba_copy_block`, `get`, `cdiv`, `collect_mamba_copy_meta`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `preprocess_mamba`，其作用域位于the module。 关键调用包括 `cleanup_mamba_state_idx`, `enumerate`, `do_mamba_copy_block`, `get`, `cdiv`, `collect_mamba_copy_meta`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `postprocess_mamba` function / `postprocess_mamba` 函数
```python
def postprocess_mamba(
    scheduler_output: SchedulerOutput,
    kv_cache_config: KVCacheConfig,
    cache_config: CacheConfig,
    input_batch: GPUInputBatch,
    requests: dict[str, CachedRequestState],
    mamba_state_idx: dict[str, int],
    num_spec_tokens: int,
    num_reqs: int,
    *,
    forward_context: dict[str, Any] | None = None,
    mamba_state_copy_funcs: tuple[MambaStateCopyFunc, ...] | None = None,
    copy_bufs: MambaCopyBuffers | None = None,
):
    """
    Post-model-execute mamba prefix-caching bookkeeping. Dispatched by
    cache_config.mamba_cache_mode:
      - "align": if a block is converted from partial to full this step,
        copy the running state into the new full block.
      - "all" + num_spec_tokens > 0: record per-request the block index of
        the last token scheduled this step, so the next step can anchor
        its in-place writes when accepted drafts leave the sequence at a
        non-block-aligned position.
    """
    if cache_config.mamba_cache_mode == "align":
        assert forward_context is not None
        assert mamba_state_copy_funcs is not None
        assert copy_bufs is not None
        num_scheduled_tokens_dict = scheduler_output.num_scheduled_tokens
        scheduled_spec_decode_tokens_dict = (
            scheduler_output.scheduled_spec_decode_tokens
        )
        num_accepted_tokens_cpu = input_batch.num_accepted_tokens_cpu
        mamba_group_ids = copy_bufs.mamba_group_ids
        mamba_spec = copy_bufs.mamba_spec
        copy_bufs.offset = 0
        for i, req_id in enumerate(input_batch.req_ids):
            req_state = requests[req_id]
            num_computed_tokens = req_state.num_computed_tokens
            num_draft_tokens = len(scheduled_spec_decode_tokens_dict.get(req_id, []))
            num_scheduled_tokens = num_scheduled_tokens_dict[req_id]
            num_accepted_tokens = num_accepted_tokens_cpu[i]
            num_tokens_running_state = (
                num_computed_tokens + num_scheduled_tokens - num_draft_tokens
            )
            new_num_computed_tokens = num_tokens_running_state + num_accepted_tokens - 1
            aligned_new_computed_tokens = (
                new_num_computed_tokens // mamba_spec.block_size * mamba_spec.block_size
            )
            # TODO: how to ensure all blocks that cache_blocks called are cached here?
            if aligned_new_computed_tokens >= num_tokens_running_state:
                accept_token_bias = (
                    aligned_new_computed_tokens - num_tokens_running_state
                )
                src_block_idx = mamba_state_idx[req_id]
                dest_block_idx = (
                    aligned_new_computed_tokens // mamba_spec.block_size - 1
                )
                collect_mamba_copy_meta(
                    copy_bufs,
                    kv_cache_config,
                    mamba_state_copy_funcs,
                    mamba_group_ids,
                    src_block_idx,
                    dest_block_idx,
                    accept_token_bias,
                    req_state,
                    forward_context,
                )
                if src_block_idx == dest_block_idx:
                    num_accepted_tokens_cpu[i] = 1
        do_mamba_copy_block(copy_bufs)
    elif cache_config.mamba_cache_mode == "all" and num_spec_tokens > 0:
        _, mamba_spec = get_mamba_groups(kv_cache_config)
        block_size = mamba_spec.block_size
        full_decode_len = 1 + num_spec_tokens
        scheduled = scheduler_output.num_scheduled_tokens
        for req_id in input_batch.req_ids[:num_reqs]:
            num_query = scheduled.get(req_id, 0)
            if num_query == full_decode_len:
                req = requests[req_id]
                seq_len = req.num_computed_tokens + num_query
                mamba_state_idx[req_id] = max(0, (seq_len - 1) // block_size)
            else:
                mamba_state_idx.pop(req_id, None)
```
**EN:** This function implements `postprocess_mamba` within the module. The docstring frames it as: Post-model-execute mamba prefix-caching bookkeeping. Key calls include `enumerate`, `do_mamba_copy_block`, `len`, `get_mamba_groups`, `get`, `collect_mamba_copy_meta`. The control flow contains 5 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `postprocess_mamba`，其作用域位于the module。 关键调用包括 `enumerate`, `do_mamba_copy_block`, `len`, `get_mamba_groups`, `get`, `collect_mamba_copy_meta`。 控制流包含 5 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `preprocess_mamba_all_specdec` function / `preprocess_mamba_all_specdec` 函数
```python
def preprocess_mamba_all_specdec(
    scheduler_output: SchedulerOutput,
    input_batch: GPUInputBatch,
    mamba_state_idx: dict[str, int],
    num_reqs: int,
    prev_last_scheduled_idx_buf: CpuGpuBuffer,
) -> None:
    cleanup_mamba_state_idx(scheduler_output, mamba_state_idx)
    np_view = prev_last_scheduled_idx_buf.np
    for i, req_id in enumerate(input_batch.req_ids[:num_reqs]):
        np_view[i] = mamba_state_idx.get(req_id, -1)
    np_view[num_reqs:].fill(-1)
    prev_last_scheduled_idx_buf.copy_to_gpu()
```
**EN:** This function implements `preprocess_mamba_all_specdec` within the module. Key calls include `cleanup_mamba_state_idx`, `enumerate`, `fill`, `copy_to_gpu`, `get`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `preprocess_mamba_all_specdec`，其作用域位于the module。 关键调用包括 `cleanup_mamba_state_idx`, `enumerate`, `fill`, `copy_to_gpu`, `get`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `batch_memcpy_kernel`: top-level helper or orchestration entry point. / `batch_memcpy_kernel`：顶层辅助函数或编排入口。
- `batch_memcpy`: top-level helper or orchestration entry point. / `batch_memcpy`：顶层辅助函数或编排入口。
- `get_mamba_groups`: top-level helper or orchestration entry point. / `get_mamba_groups`：顶层辅助函数或编排入口。
- `MambaCopyBuffers`: central class or interface in this module. / `MambaCopyBuffers`：本模块中的核心类或接口。
- `collect_mamba_copy_meta`: top-level helper or orchestration entry point. / `collect_mamba_copy_meta`：顶层辅助函数或编排入口。
- `do_mamba_copy_block`: top-level helper or orchestration entry point. / `do_mamba_copy_block`：顶层辅助函数或编排入口。
- `cleanup_mamba_state_idx`: top-level helper or orchestration entry point. / `cleanup_mamba_state_idx`：顶层辅助函数或编排入口。
- `preprocess_mamba`: top-level helper or orchestration entry point. / `preprocess_mamba`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `itertools`, `collections`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.v1.core.sched.output`, `vllm.v1.kv_cache_interface`, `vllm.v1.utils`, `vllm.v1.worker.gpu_input_batch`, `vllm.v1.worker.lora_model_runner_mixin`
