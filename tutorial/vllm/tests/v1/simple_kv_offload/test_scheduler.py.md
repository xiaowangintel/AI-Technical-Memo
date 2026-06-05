# test_scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/simple_kv_offload/test_scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for SimpleCPUOffloadScheduler. / 该文件的文档字符串表明其用途：`unit tests for simplecpuoffloadscheduler`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Unit tests for SimpleCPUOffloadScheduler."""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for SimpleCPUOffloadScheduler.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for simplecpuoffloadscheduler`。

### Imports and setup / 导入与设置 (lines 5-42)
```python
from __future__ import annotations

from dataclasses import dataclass
import torch
from vllm import SamplingParams
from vllm.config import (
    CacheConfig,
    DeviceConfig,
    KVTransferConfig,
    ModelConfig,
    SchedulerConfig,
    VllmConfig,
)
from vllm.utils.hashing import sha256
from vllm.v1.core.block_pool import BlockPool
from vllm.v1.core.kv_cache_manager import KVCacheBlocks
# ... excerpt omitted for brevity ...
    KVCacheTensor,
from vllm.v1.outputs import KVConnectorOutput
from vllm.v1.request import Request
from vllm.v1.simple_kv_offload.manager import SimpleCPUOffloadScheduler
from vllm.v1.simple_kv_offload.metadata import SimpleCPUOffloadWorkerMetadata
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `torch`. vLLM modules under test include `vllm, vllm.config, vllm.utils.hashing, vllm.v1.core.block_pool, vllm.v1.core.kv_cache_manager, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `torch`。 被测试的 vLLM 模块包括 `vllm, vllm.config, vllm.utils.hashing, vllm.v1.core.block_pool, vllm.v1.core.kv_cache_manager, ...`。

### Module state / 模块级状态 (lines 47-56)
```python
BLOCK_SIZE = 16
HEAD_SIZE = 16
NUM_KV_HEADS = 1
DTYPE = torch.float16
# bytes per block per tensor:
# block_size * num_kv_heads * head_size * 2 (K+V) * element_size
_BYTES_PER_BLOCK = BLOCK_SIZE * NUM_KV_HEADS * HEAD_SIZE * 2 * DTYPE.itemsize

# Ensure none_hash is initialized once
init_none_hash(sha256)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `BLOCK_SIZE, HEAD_SIZE, NUM_KV_HEADS, DTYPE, _BYTES_PER_BLOCK`. Shared setup calls include `init_none_hash`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`BLOCK_SIZE, HEAD_SIZE, NUM_KV_HEADS, DTYPE, _BYTES_PER_BLOCK`。 共享初始化调用包括 `init_none_hash`。

### _make_kv_cache_config (lines 64-94)
```python
def _make_kv_cache_config(
    num_blocks: int,
    num_groups: int = 1,
) -> KVCacheConfig:
    """Build a KVCacheConfig with non-empty kv_cache_tensors."""
    groups = []
    tensors = []
    for g in range(num_groups):
        layer_names = [f"layer_{g}"]
        groups.append(
            KVCacheGroupSpec(
                layer_names,
                FullAttentionSpec(
                    block_size=BLOCK_SIZE,
                    num_kv_heads=NUM_KV_HEADS,
                    head_size=HEAD_SIZE,
                    dtype=DTYPE,
                ),
            )
        )
        tensors.append(
            KVCacheTensor(
                size=_BYTES_PER_BLOCK * num_blocks,
                shared_by=layer_names,
            )
        )
    return KVCacheConfig(
        num_blocks=num_blocks,
        kv_cache_tensors=tensors,
        kv_cache_groups=groups,
    )
```
**EN:** Helper function `_make_kv_cache_config` encapsulates reusable logic for `KV cache config`. Inputs: `num_blocks, num_groups`. Key calls include `range, KVCacheConfig, groups.append, tensors.append, KVCacheGroupSpec, KVCacheTensor`.
**CN:** 辅助函数 `_make_kv_cache_config` 封装了与 `KV 缓存 config` 相关的可复用逻辑。 输入参数：`num_blocks, num_groups`。 关键调用包括 `range, KVCacheConfig, groups.append, tensors.append, KVCacheGroupSpec, KVCacheTensor`。

### _make_vllm_config (lines 97-127)
```python
def _make_vllm_config(block_size: int = BLOCK_SIZE) -> VllmConfig:
    """Minimal VllmConfig for scheduler tests (no GPU)."""
    model_config = ModelConfig(
        model="facebook/opt-125m",
        trust_remote_code=True,
        dtype="float16",
        seed=42,
    )
    scheduler_config = SchedulerConfig(
        max_num_seqs=16,
        max_num_batched_tokens=64,
        max_model_len=10000,
        enable_chunked_prefill=True,
        is_encoder_decoder=False,
    )
    cache_config = CacheConfig(
        block_size=block_size,
        gpu_memory_utilization=0.9,
        enable_prefix_caching=True,
    )
    kv_transfer_config = KVTransferConfig(
        kv_connector="SimpleCPUOffloadConnector",
        kv_role="kv_both",
    )
    return VllmConfig(
        scheduler_config=scheduler_config,
        model_config=model_config,
        cache_config=cache_config,
        kv_transfer_config=kv_transfer_config,
        device_config=DeviceConfig("cpu"),
    )
```
**EN:** Helper function `_make_vllm_config` encapsulates reusable logic for `vllm config`. Inputs: `block_size`. Key calls include `ModelConfig, SchedulerConfig, CacheConfig, KVTransferConfig, VllmConfig, DeviceConfig`.
**CN:** 辅助函数 `_make_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 输入参数：`block_size`。 关键调用包括 `ModelConfig, SchedulerConfig, CacheConfig, KVTransferConfig, VllmConfig, DeviceConfig`。

### SchedulerFixture (lines 131-138)
```python
class SchedulerFixture:
    """Bundle returned by make_scheduler for convenient access."""

    scheduler: SimpleCPUOffloadScheduler
    gpu_block_pool: BlockPool
    vllm_config: VllmConfig
    kv_cache_config: KVCacheConfig
    num_groups: int = 1
```
**EN:** Class `SchedulerFixture` groups 0 test method(s).
**CN:** 类 `SchedulerFixture` 组织了 0 个测试方法。

### make_scheduler (lines 141-173)
```python
def make_scheduler(
    num_cpu_blocks: int = 8,
    num_gpu_blocks: int = 16,
    num_groups: int = 1,
    lazy: bool = False,
) -> SchedulerFixture:
    """Build a SimpleCPUOffloadScheduler with small block pools."""
    kv_cache_config = _make_kv_cache_config(num_gpu_blocks, num_groups)
    vllm_config = _make_vllm_config()
    cpu_capacity_bytes = _BYTES_PER_BLOCK * num_cpu_blocks * num_groups

    sched = SimpleCPUOffloadScheduler(
        vllm_config=vllm_config,
        kv_cache_config=kv_cache_config,
        cpu_capacity_bytes=cpu_capacity_bytes,
        lazy_offload=lazy,
    )
    # ... excerpt omitted for brevity ...
    return SchedulerFixture(
        scheduler=sched,
        gpu_block_pool=gpu_block_pool,
        num_groups=num_groups,
```
**EN:** Helper function `make_scheduler` encapsulates reusable logic for `scheduler`. Inputs: `num_cpu_blocks, num_gpu_blocks, num_groups, lazy`. Key calls include `_make_kv_cache_config, _make_vllm_config, SimpleCPUOffloadScheduler, BlockPool, sched.bind_gpu_block_pool, SchedulerFixture`.
**CN:** 辅助函数 `make_scheduler` 封装了与 `调度器` 相关的可复用逻辑。 输入参数：`num_cpu_blocks, num_gpu_blocks, num_groups, lazy`。 关键调用包括 `_make_kv_cache_config, _make_vllm_config, SimpleCPUOffloadScheduler, BlockPool, sched.bind_gpu_block_pool, SchedulerFixture`。

### Module state / 模块级状态 (line 176)
```python
_req_counter = 0
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_req_counter`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_req_counter`。

### make_request (lines 179-203)
```python
def make_request(
    num_blocks: int = 2,
    request_id: str | None = None,
    extra_tokens: int = 1,
) -> Request:
    """Create a Request with deterministic block hashes."""
    global _req_counter
    _req_counter += 1
    if request_id is None:
        request_id = f"req-{_req_counter}"

    num_tokens = num_blocks * BLOCK_SIZE + extra_tokens
    start = _req_counter * 10000
    prompt_token_ids = list(range(start, start + num_tokens))
    sampling_params = SamplingParams(max_tokens=1)

    req = Request(
        request_id=request_id,
        prompt_token_ids=prompt_token_ids,
        sampling_params=sampling_params,
        pooling_params=None,
        mm_features=None,
        block_hasher=get_request_block_hasher(BLOCK_SIZE, sha256),
    )
    return req
```
**EN:** Helper function `make_request` encapsulates reusable logic for `request`. Inputs: `num_blocks, request_id, extra_tokens`. Key calls include `list, SamplingParams, Request, range, get_request_block_hasher`.
**CN:** 辅助函数 `make_request` 封装了与 `request` 相关的可复用逻辑。 输入参数：`num_blocks, request_id, extra_tokens`。 关键调用包括 `list, SamplingParams, Request, range, get_request_block_hasher`。

### make_scheduler_output (lines 206-263)
```python
def make_scheduler_output(
    req_id_to_num_tokens: dict[str, int],
    *,
    new_reqs: dict[str, tuple[list[int], ...]] | None = None,
    cached_req_new_blocks: dict[str, tuple[list[int], ...] | None] | None = None,
) -> SchedulerOutput:
    """Build a minimal SchedulerOutput with num_scheduled_tokens.

    Args:
        new_reqs: For first-time requests, maps req_id -> block_ids tuple.
            These are placed into ``scheduled_new_reqs`` as ``NewRequestData``.
        cached_req_new_blocks: For returning (cached) requests, maps
            req_id -> new_block_ids (incremental) or None.
            These are placed into ``scheduled_cached_reqs``.
    """
    scheduled_new_reqs: list[NewRequestData] = []
    if new_reqs:
        for req_id, block_ids in new_reqs.items():
    # ... excerpt omitted for brevity ...
    return SchedulerOutput(
        scheduled_encoder_inputs={},
        num_common_prefix_blocks=[],
        preempted_req_ids=set(),
        finished_req_ids=set(),
        free_encoder_mm_hashes=[],
    )
```
**EN:** Helper function `make_scheduler_output` encapsulates reusable logic for `scheduler output`. Inputs: `req_id_to_num_tokens`. Key calls include `SchedulerOutput, new_reqs.items, list, CachedRequestData, CachedRequestData.make_empty, scheduled_new_reqs.append`.
**CN:** 辅助函数 `make_scheduler_output` 封装了与 `调度器 output` 相关的可复用逻辑。 输入参数：`req_id_to_num_tokens`。 关键调用包括 `SchedulerOutput, new_reqs.items, list, CachedRequestData, CachedRequestData.make_empty, scheduled_new_reqs.append`。

### simulate_store_completion (lines 266-277)
```python
def simulate_store_completion(
    scheduler: SimpleCPUOffloadScheduler,
    event_idx: int,
) -> None:
    """Simulate worker reporting a store event completion."""
    output = KVConnectorOutput(
        finished_recving=set(),
        kv_connector_worker_meta=SimpleCPUOffloadWorkerMetadata(
            completed_store_events={event_idx: scheduler._expected_worker_count},
        ),
    )
    scheduler.update_connector_output(output)
```
**EN:** Helper function `simulate_store_completion` encapsulates reusable logic for `simulate store completion`. Inputs: `scheduler, event_idx`. Key calls include `KVConnectorOutput, scheduler.update_connector_output, set, SimpleCPUOffloadWorkerMetadata`.
**CN:** 辅助函数 `simulate_store_completion` 封装了与 `simulate store completion` 相关的可复用逻辑。 输入参数：`scheduler, event_idx`。 关键调用包括 `KVConnectorOutput, scheduler.update_connector_output, set, SimpleCPUOffloadWorkerMetadata`。

### simulate_load_completion (lines 280-289)
```python
def simulate_load_completion(
    scheduler: SimpleCPUOffloadScheduler,
    req_ids: set[str],
) -> None:
    """Simulate worker reporting load completions for requests."""
    output = KVConnectorOutput(
        finished_sending=set(),
        finished_recving=req_ids,
    )
    scheduler.update_connector_output(output)
```
**EN:** Helper function `simulate_load_completion` encapsulates reusable logic for `simulate load completion`. Inputs: `scheduler, req_ids`. Key calls include `KVConnectorOutput, scheduler.update_connector_output, set`.
**CN:** 辅助函数 `simulate_load_completion` 封装了与 `simulate load completion` 相关的可复用逻辑。 输入参数：`scheduler, req_ids`。 关键调用包括 `KVConnectorOutput, scheduler.update_connector_output, set`。

### get_cpu_free_blocks (lines 292-294)
```python
def get_cpu_free_blocks(scheduler: SimpleCPUOffloadScheduler) -> int:
    """Return number of free CPU blocks."""
    return scheduler.cpu_block_pool.get_num_free_blocks()
```
**EN:** Helper function `get_cpu_free_blocks` encapsulates reusable logic for `CPU free blocks`. Inputs: `scheduler`. Key calls include `cpu_block_pool.get_num_free_blocks`.
**CN:** 辅助函数 `get_cpu_free_blocks` 封装了与 `cpu free blocks` 相关的可复用逻辑。 输入参数：`scheduler`。 关键调用包括 `cpu_block_pool.get_num_free_blocks`。

### _allocate_gpu_blocks (lines 297-320)
```python
def _allocate_gpu_blocks(
    gpu_block_pool: BlockPool,
    request: Request,
    num_blocks: int,
    group_id: int = 0,
) -> list:
    """Allocate GPU blocks, cache them with hashes, return block list.

    Mimics what KVCacheManager does: allocate blocks from pool, then
    register them in the prefix cache via cache_full_blocks so that
    re-allocation properly evicts stale hashes.
    """
    blocks = gpu_block_pool.get_new_blocks(num_blocks)
    num_full = min(num_blocks, len(request.block_hashes))
    if num_full > 0:
        gpu_block_pool.cache_full_blocks(
            request=request,
            blocks=blocks,
            num_cached_blocks=0,
            num_full_blocks=num_full,
            block_size=BLOCK_SIZE,
            kv_cache_group_id=group_id,
        )
    return blocks
```
**EN:** Helper function `_allocate_gpu_blocks` encapsulates reusable logic for `allocate GPU blocks`. Inputs: `gpu_block_pool, request, num_blocks, group_id`. Key calls include `gpu_block_pool.get_new_blocks, min, len, gpu_block_pool.cache_full_blocks`.
**CN:** 辅助函数 `_allocate_gpu_blocks` 封装了与 `allocate gpu blocks` 相关的可复用逻辑。 输入参数：`gpu_block_pool, request, num_blocks, group_id`。 关键调用包括 `gpu_block_pool.get_new_blocks, min, len, gpu_block_pool.cache_full_blocks`。

### _alloc_and_register (lines 323-346)
```python
def _alloc_and_register(
    fix: SchedulerFixture,
    request: Request,
    num_blocks: int,
    *,
    confirmed: bool = True,
    group_id: int = 0,
) -> KVCacheBlocks:
    """Allocate GPU blocks and return KVCacheBlocks.

    Block IDs are no longer registered in a mock KVCacheManager; instead
    tests pass them through ``make_scheduler_output`` so that
    ``yield_req_data`` can pick them up.

    If ``confirmed`` is True, advance ``request.num_computed_tokens`` to simulate
    the scheduler's ``_update_after_schedule`` from a prior step.
    """
    gpu_blocks = _allocate_gpu_blocks(
        fix.gpu_block_pool, request, num_blocks, group_id=group_id
    )
    kv_blocks = KVCacheBlocks(blocks=(gpu_blocks,))
    if confirmed:
        request.num_computed_tokens = num_blocks * BLOCK_SIZE
    return kv_blocks
```
**EN:** Helper function `_alloc_and_register` encapsulates reusable logic for `alloc and register`. Inputs: `fix, request, num_blocks`. Key calls include `_allocate_gpu_blocks, KVCacheBlocks`.
**CN:** 辅助函数 `_alloc_and_register` 封装了与 `alloc and register` 相关的可复用逻辑。 输入参数：`fix, request, num_blocks`。 关键调用包括 `_allocate_gpu_blocks, KVCacheBlocks`。

### test_eager_store_and_load_roundtrip (lines 352-401)
```python
def test_eager_store_and_load_roundtrip() -> None:
    """Eager mode: store blocks on compute, complete store, verify cache hit."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: block_ids},
    )
    meta = sched.build_connector_meta(sched_out)
    assert meta.store_event >= 0, "Expected a store event to be scheduled"
    # ... excerpt omitted for brevity ...
    assert len(meta.store_gpu_blocks) > 0
    assert len(meta.store_cpu_blocks) == len(meta.store_gpu_blocks)
    assert hit_tokens == num_blocks * BLOCK_SIZE
    assert is_async is True
        new_reqs={req2.request_id: block_ids2},
    meta2 = sched.build_connector_meta(sched_out2)
    assert meta2.load_event >= 0, "Expected a load event to be assigned"
    assert len(meta2.load_gpu_blocks) > 0
    assert len(meta2.load_cpu_blocks) == len(meta2.load_gpu_blocks)
```
**EN:** Test case covering `eager store and load roundtrip`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `eager store and load roundtrip` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 8 个显式断言。

### test_max_hit_len_cap_drops_last_full_block (lines 408-438)
```python
def test_max_hit_len_cap_drops_last_full_block() -> None:
    """When num_tokens is an exact multiple of BLOCK_SIZE, the manager's
    ``max_hit_len = num_tokens - 1`` cap forces ``find_longest_cache_hit`` to
    drop the final block (since ``max_length // block_size`` rounds down).
    """
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    req = make_request(num_blocks=num_blocks, extra_tokens=0)
    assert req.num_tokens == num_blocks * BLOCK_SIZE

    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: kv_blocks.get_block_ids()},
    )
    meta = sched.build_connector_meta(sched_out)
    simulate_store_completion(sched, meta.store_event)

    req2 = Request(
        request_id="req-cap-boundary",
        prompt_token_ids=req.prompt_token_ids,
        sampling_params=req.sampling_params,
        pooling_params=None,
        mm_features=None,
        block_hasher=req._block_hasher,
    )
    hit_tokens, _ = sched.get_num_new_matched_tokens(req2, num_computed_tokens=0)
    assert hit_tokens == (num_blocks - 1) * BLOCK_SIZE
```
**EN:** Test case covering `max hit len cap drops last full block`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, make_scheduler_output, sched.build_connector_meta`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `max hit len cap drops last full block` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, make_scheduler_output, sched.build_connector_meta`。 代码主体包含 2 个显式断言。

### _flush_old_blocks_to_lru_head (lines 444-458)
```python
def _flush_old_blocks_to_lru_head(
    gpu_pool: BlockPool,
    num_filler_blocks: int,
) -> list:
    """Allocate filler blocks so that previously-freed (hashed) blocks migrate
    to the LRU head of the free queue.  Returns the filler blocks (caller must
    free them later to restore pool capacity).

    In a real engine the same thing happens naturally: after one request
    finishes and frees its blocks, subsequent requests allocate from the LRU
    head, consuming the unhashed blocks and leaving the old hashed blocks at
    the front of the queue.
    """
    fillers = gpu_pool.get_new_blocks(num_filler_blocks)
    return fillers
```
**EN:** Helper function `_flush_old_blocks_to_lru_head` encapsulates reusable logic for `flush old blocks to lru head`. Inputs: `gpu_pool, num_filler_blocks`. Key calls include `gpu_pool.get_new_blocks`.
**CN:** 辅助函数 `_flush_old_blocks_to_lru_head` 封装了与 `flush old blocks to lru head` 相关的可复用逻辑。 输入参数：`gpu_pool, num_filler_blocks`。 关键调用包括 `gpu_pool.get_new_blocks`。

### test_lazy_store_and_load_roundtrip (lines 461-524)
```python
def test_lazy_store_and_load_roundtrip() -> None:
    """Lazy mode: schedule a request, finish it so its hashed blocks are freed,
    then schedule new requests so the old blocks migrate to the LRU head.
    The lazy scanner offloads them to CPU.  Re-scheduling the old request
    triggers a CPU cache hit + load.

    GPU pool: 8 blocks (7 usable).  _target_free = ceil(64/16) = 4.
    """
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=8, lazy=True)
    sched = fix.scheduler
    gpu_pool = fix.gpu_block_pool
    num_blocks = 2
    # --- Step 1: Schedule req_old, compute, and finish ---
    req_old = make_request(num_blocks=num_blocks)
    gpu_blocks_old = _allocate_gpu_blocks(gpu_pool, req_old, num_blocks, group_id=0)
    gpu_pool.free_blocks(gpu_blocks_old)
    # ... excerpt omitted for brevity ...
    assert meta.store_event >= 0, "Expected lazy store to offload old blocks"
    assert len(meta.store_gpu_blocks) == num_blocks
    expected_hit = num_blocks * BLOCK_SIZE
    assert hit_tokens == expected_hit, (
        f"Expected {expected_hit} hit tokens, got {hit_tokens}"
    assert is_async is True
    )
    sched_out2 = make_scheduler_output({req_old2.request_id: 1})
    meta2 = sched.build_connector_meta(sched_out2)
    assert meta2.load_event >= 0, "Expected a load event to be assigned"
    assert len(meta2.load_gpu_blocks) > 0
```
**EN:** Test case covering `lazy store and load roundtrip`. It exercises `make_scheduler, make_request, _allocate_gpu_blocks, gpu_pool.free_blocks, _flush_old_blocks_to_lru_head, make_scheduler_output`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `lazy store and load roundtrip` 的测试用例。 该测试会调用 `make_scheduler, make_request, _allocate_gpu_blocks, gpu_pool.free_blocks, _flush_old_blocks_to_lru_head, make_scheduler_output`。 代码主体包含 6 个显式断言。

### test_eager_duplicate_store_skipped (lines 530-573)
```python
def test_eager_duplicate_store_skipped() -> None:
    """Eager: storing the same block hashes twice should not allocate new CPU blocks."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: block_ids},
    )
    meta1 = sched.build_connector_meta(sched_out)
    assert meta1.store_event >= 0
    # ... excerpt omitted for brevity ...
    meta2 = sched.build_connector_meta(sched_out2)
    if meta2.store_event >= 0:
        assert len(meta2.store_cpu_blocks) == 0, (
            "Expected no new CPU blocks for duplicate hashes"
        )
    assert get_cpu_free_blocks(sched) == cpu_free_after_first
```
**EN:** Test case covering `eager duplicate store skipped`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `eager duplicate store skipped` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 3 个显式断言。

### test_eager_in_flight_store_dedup_across_steps (lines 579-638)
```python
def test_eager_in_flight_store_dedup_across_steps() -> None:
    """Eager: a second request sharing a prefix with an in-flight store
    must not re-offload the same GPU blocks before completion lands.

    Simulates a GPU prefix-cache hit by reusing the first request's
    GPU block IDs in the second scheduler step, which is the path the
    real scheduler takes when two requests share a prefix.
    """
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler
    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
    # ... excerpt omitted for brevity ...
    assert meta1.store_event >= 0
    assert len(meta1.store_cpu_blocks) == num_blocks
    assert sched._in_flight_store_gpu_blocks == set(meta1.store_gpu_blocks)
        assert len(meta2.store_cpu_blocks) == 0, (
    assert get_cpu_free_blocks(sched) == cpu_free_after_first, (
        "store is still in-flight"
    )
    # After completion, the in-flight set is cleared.
    simulate_store_completion(sched, meta1.store_event)
    assert sched._in_flight_store_gpu_blocks == set()
```
**EN:** Test case covering `eager in flight store dedup across steps`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `eager in flight store dedup across steps` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 6 个显式断言。

### test_lazy_duplicate_store_skipped (lines 644-692)
```python
def test_lazy_duplicate_store_skipped() -> None:
    """Lazy: blocks already offloaded to CPU should not be offloaded again.

    Same pattern as the lazy roundtrip: flush old blocks to LRU head, offload,
    then repeat with the same hashes and verify no new CPU allocation.
    """
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=8, lazy=True)
    sched = fix.scheduler
    gpu_pool = fix.gpu_block_pool
    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    # Schedule + finish → hashed blocks in free queue
    gpu_blocks = _allocate_gpu_blocks(gpu_pool, req, num_blocks, group_id=0)
    gpu_pool.free_blocks(gpu_blocks)
    # Flush old blocks to LRU head, then trigger lazy offload.
    # ... excerpt omitted for brevity ...
    assert meta1.store_event >= 0
    # Either no store event, or zero new CPU blocks (already cached).
    if meta2.store_event >= 0:
        assert len(meta2.store_cpu_blocks) == 0, (
            "Expected no new CPU blocks for duplicate hashes"
        )
    assert get_cpu_free_blocks(sched) == cpu_free_after_first
```
**EN:** Test case covering `lazy duplicate store skipped`. It exercises `make_scheduler, make_request, _allocate_gpu_blocks, gpu_pool.free_blocks, _flush_old_blocks_to_lru_head, sched.build_connector_meta`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `lazy duplicate store skipped` 的测试用例。 该测试会调用 `make_scheduler, make_request, _allocate_gpu_blocks, gpu_pool.free_blocks, _flush_old_blocks_to_lru_head, sched.build_connector_meta`。 代码主体包含 3 个显式断言。

### test_lru_eviction_order (lines 698-785)
```python
def test_lru_eviction_order() -> None:
    """With limited CPU space, oldest blocks should be evicted first.

    CPU block pool: num_cpu_blocks=5 -> 4 free usable blocks (1 taken by null_block).
    After storing 4 blocks (2 req_a + 2 req_b), all free slots are occupied by
    cached blocks (ref_cnt=0, in hash map).  When 2 more are stored (req_c),
    2 LRU blocks from req_a get evicted from the cache to make room.
    """
    # 5 total = 4 usable (null_block takes 1), filling exactly with 4 blocks
    fix = make_scheduler(num_cpu_blocks=5, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler
    # Fill CPU with 4 blocks: 2 requests x 2 blocks (in LRU insertion order)
    req_a = make_request(num_blocks=2)
    req_b = make_request(num_blocks=2)
    kv_a = _alloc_and_register(fix, req_a, 2)
    kv_b = _alloc_and_register(fix, req_b, 2)
    # ... excerpt omitted for brevity ...
    assert meta.store_event >= 0
        assert (
    assert meta2.store_event >= 0
        assert cached is None, f"req_a block {i} should have been evicted"
        assert cached is not None, f"req_b block {i} should still be cached"
    for i, bhash in enumerate(req_c.block_hashes[:2]):
        bhash_with_group = make_block_hash_with_group_id(bhash, 0)
        cache_map = sched.cpu_block_pool.cached_block_hash_to_block
        cached = cache_map.get_one_block(bhash_with_group)
        assert cached is not None, f"req_c block {i} should still be cached"
```
**EN:** Test case covering `lru eviction order`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_a.get_block_ids, kv_b.get_block_ids`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `lru eviction order` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_a.get_block_ids, kv_b.get_block_ids`。 代码主体包含 7 个显式断言。

### test_touched_blocks_survive_eviction (lines 791-854)
```python
def test_touched_blocks_survive_eviction() -> None:
    """Touching CPU blocks updates their LRU position, protecting them from eviction."""
    # 5 total = 4 usable (null_block takes 1)
    fix = make_scheduler(num_cpu_blocks=5, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    # Fill CPU with 4 blocks (req_a: 2, req_b: 2) in LRU order
    req_a = make_request(num_blocks=2)
    req_b = make_request(num_blocks=2)
    kv_a = _alloc_and_register(fix, req_a, 2)
    kv_b = _alloc_and_register(fix, req_b, 2)
    sched.update_state_after_alloc(req_a, kv_a, num_external_tokens=0)
    sched.update_state_after_alloc(req_b, kv_b, num_external_tokens=0)
    ids_a = kv_a.get_block_ids()
    ids_b = kv_b.get_block_ids()
    sched_out = make_scheduler_output(
    # ... excerpt omitted for brevity ...
        assert cached_blk is not None
        assert cached is None, f"req_b block {i} should have been evicted (it was LRU)"
    for i, bhash in enumerate(req_a.block_hashes[:2]):
        bhash_with_group = make_block_hash_with_group_id(bhash, 0)
        cached = cpu_pool.cached_block_hash_to_block.get_one_block(bhash_with_group)
        assert cached is not None, f"req_a block {i} should survive (was touched/MRU)"
```
**EN:** Test case covering `touched blocks survive eviction`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_a.get_block_ids, kv_b.get_block_ids`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `touched blocks survive eviction` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_a.get_block_ids, kv_b.get_block_ids`。 代码主体包含 3 个显式断言。

### test_preemption_no_cpu_block_leak (lines 860-912)
```python
def test_preemption_no_cpu_block_leak() -> None:
    """request_finished during in-flight load defers cleanup;
    completes after load done."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    # First: store blocks to CPU
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: block_ids},
    )
    meta = sched.build_connector_meta(sched_out)
    # ... excerpt omitted for brevity ...
    assert hit_tokens > 0
    assert meta2.load_event >= 0
    assert req2.request_id in sched._reqs_to_load
    assert sched._reqs_to_load[req2.request_id].finished is True
    # Now simulate load completion -> cleanup fires
    simulate_load_completion(sched, {req2.request_id})
    assert req2.request_id not in sched._reqs_to_load
```
**EN:** Test case covering `preemption no CPU block leak`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `preemption no cpu block leak` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 5 个显式断言。

### test_eager_store_preemption_cleanup (lines 918-949)
```python
def test_eager_store_preemption_cleanup() -> None:
    """In eager mode, finishing a request during in-flight store defers cleanup."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)

    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: block_ids},
    )
    meta = sched.build_connector_meta(sched_out)
    store_event = meta.store_event
    assert store_event >= 0

    # The request gets store_events populated
    assert req.request_id in sched._reqs_to_store
    store_state = sched._reqs_to_store[req.request_id]
    assert store_event in store_state.store_events

    # Finish request while store still in-flight -> deferred
    sched.request_finished(req, block_ids=[])
    assert req.request_id in sched._reqs_to_store
    assert sched._reqs_to_store[req.request_id].finished is True

    # Simulate store completion -> deferred cleanup fires
    simulate_store_completion(sched, store_event)
    assert req.request_id not in sched._reqs_to_store
```
**EN:** Test case covering `eager store preemption cleanup`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `eager store preemption cleanup` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 6 个显式断言。

### test_inflight_finish_deferred_cleanup (lines 955-1005)
```python
def test_inflight_finish_deferred_cleanup() -> None:
    """Store, then start a load, request_finished defers,
    load completion fires cleanup."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 2
    # Store
    req = make_request(num_blocks=num_blocks)
    kv_blocks = _alloc_and_register(fix, req, num_blocks)
    sched.update_state_after_alloc(req, kv_blocks, num_external_tokens=0)
    block_ids = kv_blocks.get_block_ids()
    sched_out = make_scheduler_output(
        {req.request_id: num_blocks * BLOCK_SIZE},
        new_reqs={req.request_id: block_ids},
    )
    meta = sched.build_connector_meta(sched_out)
    # ... excerpt omitted for brevity ...
    assert hit_tokens > 0
    assert meta2.load_event >= 0
    sched.request_finished(req2, block_ids=[])
    assert req2.request_id in sched._reqs_to_load
    # Simulate load completion -> request removed
    simulate_load_completion(sched, {req2.request_id})
    assert req2.request_id not in sched._reqs_to_load
```
**EN:** Test case covering `inflight finish deferred cleanup`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `inflight finish deferred cleanup` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 4 个显式断言。

### test_multi_group_null_blocks_skipped (lines 1011-1082)
```python
def test_multi_group_null_blocks_skipped() -> None:
    """Null GPU blocks (no block_hash) must not appear in store or load pairs.

    In eager store mode, _prepare_eager_store_specs skips blocks whose
    block_hash is None (null blocks have no hash). We verify this by mixing
    real hashed blocks with unhashed (null-like) blocks in a single group and
    checking that only real blocks appear in the store list.
    """
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, num_groups=1, lazy=False)
    sched = fix.scheduler
    gpu_pool = fix.gpu_block_pool
    num_blocks = 2
    req = make_request(num_blocks=num_blocks)
    # Allocate real blocks (with hashes) and use the null_block as a placeholder
    gpu_blocks = _allocate_gpu_blocks(gpu_pool, req, num_blocks, group_id=0)
    null_block = gpu_pool.null_block
    # ... excerpt omitted for brevity ...
    assert null_block_id not in meta.store_gpu_blocks, (
    assert len(meta.store_gpu_blocks) == 1
    assert gpu_blocks[0].block_id in meta.store_gpu_blocks
    assert meta.store_event >= 0
    assert hit_tokens == BLOCK_SIZE
    assert is_async is True
    meta2 = sched.build_connector_meta(sched_out2)
    # Null block's ID should NOT appear in load_gpu_blocks
    assert null_block_id not in meta2.load_gpu_blocks, (
        f"Null block id {null_block_id} should not appear in load transfer pairs"
    )
```
**EN:** Test case covering `multi group null blocks skipped`. It exercises `make_scheduler, make_request, _allocate_gpu_blocks, KVCacheBlocks, sched.update_state_after_alloc, kv_blocks.get_block_ids`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `multi group null blocks skipped` 的测试用例。 该测试会调用 `make_scheduler, make_request, _allocate_gpu_blocks, KVCacheBlocks, sched.update_state_after_alloc, kv_blocks.get_block_ids`。 代码主体包含 7 个显式断言。

### test_chunked_prefill_reads_live_block_ids (lines 1088-1139)
```python
def test_chunked_prefill_reads_live_block_ids() -> None:
    """With chunked prefill, block IDs accumulate across scheduler steps.
    _prepare_eager_store_specs reads block IDs from scheduler_output via
    yield_req_data, so the store should reflect the updated (larger) block
    list, not a stale snapshot."""
    fix = make_scheduler(num_cpu_blocks=8, num_gpu_blocks=16, lazy=False)
    sched = fix.scheduler

    num_blocks = 4
    req = make_request(num_blocks=num_blocks)
    # First chunk: allocate 2 blocks
    kv_blocks_first = _alloc_and_register(fix, req, 2)
    sched.update_state_after_alloc(req, kv_blocks_first, num_external_tokens=0)
    assert req.request_id in sched._reqs_to_store
    # Should still be exactly 1 entry in _reqs_to_store
    assert list(sched._reqs_to_store.keys()).count(req.request_id) == 1
    # ... excerpt omitted for brevity ...
    assert meta1.store_event >= 0
    assert len(meta1.store_gpu_blocks) == 2
        cached_req_new_blocks={req.request_id: new_block_ids},
    )
    meta2 = sched.build_connector_meta(sched_out2)
    assert meta2.store_event >= 0
    # Only the 2 NEW blocks should be stored (first 2 already done)
    assert len(meta2.store_gpu_blocks) == 2
```
**EN:** Test case covering `chunked prefill reads live block ids`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks_first.get_block_ids, make_scheduler_output`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `chunked prefill reads live block ids` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks_first.get_block_ids, make_scheduler_output`。 代码主体包含 7 个显式断言。

### test_partial_gpu_prefix_plus_cpu_load (lines 1145-1243)
```python
def test_partial_gpu_prefix_plus_cpu_load() -> None:
    """When GPU has a prefix cache hit for the first N blocks, CPU has a
    hit for the next M blocks, and there are P new blocks needing fresh
    compute, the block layout is:

        | comp (N) | ext_comp (M) | new (P) |
    External blocks sit in the middle — not at the beginning or end.
    The load path must target hashes at positions [N, N+M).
    Request: 6 blocks (0..5).
    - Store all 6 to CPU.
    - New request: GPU prefix cache hits blocks 0,1 (hashed).
      CPU hits blocks 2,3. Blocks 4,5 are new (need compute).
    - update_state_after_alloc receives 6 GPU blocks:
      [0,1] hashed (comp), [2,3] unhashed (ext_comp), [4,5] unhashed (new).
    - Load must target hash positions 2,3.
    """
    # ... excerpt omitted for brevity ...
    assert meta.store_event >= 0
    assert hit_tokens == num_cpu_hit_blocks * BLOCK_SIZE, (
    assert is_async is True
    assert meta2.load_event >= 0, "Expected a load event for partial GPU + CPU hit"
    assert len(meta2.load_gpu_blocks) == num_ext_blocks
    assert len(meta2.load_cpu_blocks) == num_ext_blocks
    # not the comp blocks (0,1) or new blocks (4,5).
    ext_block_ids = [b.block_id for b in gpu_ext_and_new[:num_ext_blocks]]
    for bid in meta2.load_gpu_blocks:
        assert bid in ext_block_ids, (
            f"Load GPU block {bid} should be an ext_comp block, not a comp or new block"
        )
```
**EN:** Test case covering `partial GPU prefix plus CPU load`. It exercises `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `partial gpu prefix plus cpu load` 的测试用例。 该测试会调用 `make_scheduler, make_request, _alloc_and_register, sched.update_state_after_alloc, kv_blocks.get_block_ids, make_scheduler_output`。 代码主体包含 7 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `torch`.
- **CN:** 外部库：`torch`。
- **EN:** vLLM modules under test: `vllm, vllm.config, vllm.utils.hashing, vllm.v1.core.block_pool, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.kv_cache_interface, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.config, vllm.utils.hashing, vllm.v1.core.block_pool, vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.v1.core.sched.output, vllm.v1.kv_cache_interface, ...`。
- **EN:** Standard-library support: `__future__, dataclasses`.
- **CN:** 标准库支持：`__future__, dataclasses`。
