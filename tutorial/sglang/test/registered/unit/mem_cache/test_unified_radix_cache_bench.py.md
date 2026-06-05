# test_unified_radix_cache_bench.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_unified_radix_cache_bench.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates unified radix cache bench behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 unified radix cache bench 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""Large-scale benchmark + fuzz correctness tests for UnifiedRadixCache.

Usage (standalone):
    bench: python3 test/registered/unit/mem_cache/test_unified_radix_cache_bench.py --num-seqs 5000 --verify --components mamba legacy-mamba swa legacy-swa
    CI Test: python -m pytest test/registered/unit/mem_cache/test_unified_radix_cache_bench.py -v -s
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-39: module imports and dependencies / 模块导入与依赖
```python
import argparse
import gc
import logging
import random
import statistics
import time
import unittest
from contextlib import contextmanager
from dataclasses import dataclass
from typing import Callable

import torch

from sglang.srt.configs.mamba_utils import Mamba2CacheParams, Mamba2StateShape
from sglang.srt.environ import envs
from sglang.srt.mem_cache.allocator import TokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import (
    DecLockRefParams,
    EvictParams,
    InsertParams,
    MatchPrefixParams,
)
from sglang.srt.mem_cache.cache_init_params import CacheInitParams
from sglang.srt.mem_cache.mamba_radix_cache import MambaRadixCache
from sglang.srt.mem_cache.memory_pool import HybridLinearKVPool, HybridReqToTokenPool
from sglang.srt.mem_cache.radix_cache import RadixKey
from sglang.srt.mem_cache.swa_radix_cache import SWARadixCache
from sglang.srt.mem_cache.unified_cache_components.tree_component import ComponentType
from sglang.srt.mem_cache.unified_radix_cache import UnifiedRadixCache
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `argparse`, `gc`, `logging`, `random`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `argparse`, `gc`, `logging`, `random`。

### Lines 41-57: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=25, stage="base-b", runner_config="1-gpu-small")

# ---------------------------------------------------------------------------
# Constants
# ---------------------------------------------------------------------------
_HEAD_NUM = 2
_HEAD_DIM = 16
_NUM_LAYERS = 8
_GLOBAL_INTERVAL = 4
_DTYPE = torch.bfloat16
_SWA_WINDOW_SIZE = 128

_BENCH_NUM_SEQS = 5000
_BENCH_KV_SIZE = 500_000
_BENCH_CHUNK_LEN = 256

_DEFAULT_COMPONENTS = (ComponentType.FULL, ComponentType.MAMBA)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 60-68: function suppress logs / 函数 suppress logs
```python
@contextmanager
def _suppress_logs():
    root = logging.getLogger()
    prev = root.level
    root.setLevel(logging.WARNING)
    try:
        yield
    finally:
        root.setLevel(prev)
```
**EN:** This block implements `_suppress_logs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_suppress_logs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 71-72: function full attention layer ids / 函数 full attention layer ids
```python
def _full_attention_layer_ids():
    return list(range(_GLOBAL_INTERVAL - 1, _NUM_LAYERS, _GLOBAL_INTERVAL))
```
**EN:** This block implements `_full_attention_layer_ids` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_full_attention_layer_ids`，承担模块行为中的一个聚焦逻辑片段。

### Lines 75-77: function non full layer ids / 函数 non full layer ids
```python
def _non_full_layer_ids():
    full = set(_full_attention_layer_ids())
    return [i for i in range(_NUM_LAYERS) if i not in full]
```
**EN:** This block implements `_non_full_layer_ids` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_non_full_layer_ids`，承担模块行为中的一个聚焦逻辑片段。

### Lines 78-82: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Sequence generator
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 83-117: function gen random sequences / 函数 gen random sequences
```python
def gen_random_sequences(
    num_seqs: int = 2000,
    chunk_len: int = 256,
    vocab_size: int = 32000,
    seed: int = 42,
) -> list[list[int]]:
    """Generate *num_seqs* token sequences with tree-like prefix sharing.

    Phase 1 (50%): chain growth — each new seq extends a random existing one.
    Phase 2 (50%): fan-out burst — multiple children from the same parent.
    """
    rng = random.Random(seed)
    root_prefix = [rng.randint(1, vocab_size) for _ in range(max(1, chunk_len // 4))]
    sequences: list[list[int]] = [root_prefix[:]]

    # Phase 1: chain growth
    for _ in range(num_seqs // 2):
        parent = rng.choice(sequences)
        sequences.append(
            parent + [rng.randint(1, vocab_size)] * rng.randint(1, chunk_len)
        )

    # Phase 2: fan-out burst
    remaining = num_seqs - num_seqs // 2
    while remaining > 0:
        fan = min(rng.randint(2, 10), remaining)
        parent = rng.choice(sequences)
        for _ in range(fan):
            sequences.append(
                parent + [rng.randint(1, vocab_size)] * rng.randint(1, chunk_len)
            )
        remaining -= fan

    rng.shuffle(sequences)
    return sequences
```
**EN:** Generate *num_seqs* token sequences with tree-like prefix sharing. This block implements `gen_random_sequences` and captures one focused piece of the module's behavior.
**CN:** Generate *num_seqs* token sequences with tree-like prefix sharing. 该代码块实现 `gen_random_sequences`，承担模块行为中的一个聚焦逻辑片段。

### Lines 118-122: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Cache factory
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 123-202: helper routine create bench cache (part 1/2) / 辅助流程 create bench cache（第 1/2 部分）
```python
def create_bench_cache(
    kv_size,
    max_num_reqs,
    max_context_len,
    components,
    page_size=1,
    tree_cls=None,
    sliding_window_size=_SWA_WINDOW_SIZE,
):
    """Create cache.  Returns (tree, allocator, req_to_token_pool, make_req)."""
    device = get_device()
    has_mamba = ComponentType.MAMBA in components
    has_swa = ComponentType.SWA in components

    mamba2_cache_params = None
    if has_mamba:
        with envs.SGLANG_MAMBA_SSM_DTYPE.override("bfloat16"):
            shape = Mamba2StateShape.create(
                tp_world_size=1,
                intermediate_size=256,
                n_groups=1,
                num_heads=2,
                head_dim=16,
                state_size=16,
                conv_kernel=4,
            )
            mamba2_cache_params = Mamba2CacheParams(
                shape=shape, layers=_non_full_layer_ids()
            )

    # --- req_to_token pool ---
    if has_mamba:
        req_to_token_pool = HybridReqToTokenPool(
            size=max_num_reqs,
            mamba_size=max(max_num_reqs * 2, 200),
            mamba_spec_state_size=max_num_reqs,
            max_context_len=max_context_len,
            device=device,
            enable_memory_saver=False,
            cache_params=mamba2_cache_params,
            mamba_layer_ids=_non_full_layer_ids(),
            enable_mamba_extra_buffer=(page_size > 1),
            speculative_num_draft_tokens=3,
        )
    else:
        from sglang.srt.mem_cache.memory_pool import ReqToTokenPool

        req_to_token_pool = ReqToTokenPool(
            size=max_num_reqs,
            max_context_len=max_context_len,
            device=device,
            enable_memory_saver=False,
        )

    # --- KV pool + allocator ---
    if has_swa:
        from sglang.srt.mem_cache.swa_memory_pool import (
            SWAKVPool,
            SWATokenToKVPoolAllocator,
        )

        pool = SWAKVPool(
            size=kv_size,
            size_swa=kv_size,
            page_size=page_size,
            dtype=_DTYPE,
            head_num=_HEAD_NUM,
            head_dim=_HEAD_DIM,
            swa_attention_layer_ids=_non_full_layer_ids(),
            full_attention_layer_ids=_full_attention_layer_ids(),
            enable_kvcache_transpose=False,
            device=device,
        )
        allocator = SWATokenToKVPoolAllocator(
            size=kv_size,
            size_swa=kv_size,
            page_size=page_size,
            dtype=_DTYPE,
            device=device,
            kvcache=pool,
```
**EN:** Create cache. This helper encapsulates `create_bench_cache` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** Create cache. 该辅助函数封装了 `create_bench_cache`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 203-256: helper routine create bench cache (part 2/2) / 辅助流程 create bench cache（第 2/2 部分）
```python
            need_sort=False,
        )
    else:
        pool = HybridLinearKVPool(
            size=kv_size,
            dtype=_DTYPE,
            page_size=page_size,
            head_num=_HEAD_NUM,
            head_dim=_HEAD_DIM,
            full_attention_layer_ids=_full_attention_layer_ids(),
            enable_kvcache_transpose=False,
            device=device,
            enable_memory_saver=False,
            mamba_pool=req_to_token_pool.mamba_pool if has_mamba else None,
        )
        allocator = TokenToKVPoolAllocator(
            size=kv_size,
            dtype=_DTYPE,
            device=device,
            kvcache=pool,
            need_sort=False,
        )

    # --- tree ---
    if tree_cls is None:
        tree_cls = UnifiedRadixCache
    tree = tree_cls(
        params=CacheInitParams(
            req_to_token_pool=req_to_token_pool,
            token_to_kv_pool_allocator=allocator,
            page_size=page_size,
            disable=False,
            tree_components=components if tree_cls is UnifiedRadixCache else None,
            sliding_window_size=sliding_window_size if has_swa else None,
        )
    )

    _rid = [0]

    def make_req():
        from sglang.srt.managers.schedule_batch import Req
        from sglang.srt.sampling.sampling_params import SamplingParams

        req = Req(
            rid=_rid[0],
            origin_input_text="",
            origin_input_ids=[],
            sampling_params=SamplingParams(temperature=0, max_new_tokens=1),
        )
        _rid[0] += 1
        req_to_token_pool.alloc([req])
        return req

    return tree, allocator, req_to_token_pool, make_req
```
**EN:** Create cache. This helper encapsulates `create_bench_cache` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** Create cache. 该辅助函数封装了 `create_bench_cache`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 257-261: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Shared bench environment + helpers
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 262-263: class _Env declaration / 类 _Env 声明
```python
@dataclass
class _Env:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 264-272: class-level constants and configuration for `_Env` / 类级常量与配置
```python
    tree: object
    alloc: object
    rtp: object
    make_req: Callable
    seqs: list
    has_mamba: bool
    has_swa: bool
    page_size: int
    avg_tokens: int
```
**EN:** This block defines shared names such as `tree`, `alloc`, `rtp`, `make_req`, `seqs`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `tree`, `alloc`, `rtp`, `make_req`, `seqs` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 275-301: function make env / 函数 make env
```python
def _make_env(num_seqs, chunk_len, kv_size, components, tree_cls=None, page_size=1):
    """Create sequences + cache, return shared _Env."""
    if components is None:
        components = _DEFAULT_COMPONENTS
    seqs = gen_random_sequences(num_seqs=num_seqs, chunk_len=chunk_len)
    max_seq_len = max(len(s) for s in seqs)
    avg_tokens = sum(len(s) for s in seqs) // len(seqs)
    with _suppress_logs():
        tree, alloc, rtp, make_req = create_bench_cache(
            kv_size=kv_size,
            max_num_reqs=num_seqs + 100,
            max_context_len=max_seq_len + 10,
            components=components,
            page_size=page_size,
            tree_cls=tree_cls,
        )
    return _Env(
        tree,
        alloc,
        rtp,
        make_req,
        seqs,
        ComponentType.MAMBA in components,
        ComponentType.SWA in components,
        page_size,
        avg_tokens,
    )
```
**EN:** Create sequences + cache, return shared _Env. This block implements `_make_env` and captures one focused piece of the module's behavior.
**CN:** Create sequences + cache, return shared _Env. 该代码块实现 `_make_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 304-317: function alloc / 函数 alloc
```python
def _alloc(env, n):
    if env.has_swa and env.page_size > 1:
        ps = env.page_size
        aligned = ((n + ps - 1) // ps) * ps
        if aligned > env.alloc.full_attn_allocator.available_size():
            return None
        if aligned > env.alloc.swa_attn_allocator.available_size():
            return None
        full_indices = env.alloc.full_attn_allocator.alloc(aligned)
        swa_indices = env.alloc.swa_attn_allocator.alloc(aligned)
        assert full_indices is not None and swa_indices is not None
        env.alloc.full_to_swa_index_mapping[full_indices] = swa_indices
        return full_indices[:n]
    return env.alloc.alloc(n)
```
**EN:** This block implements `_alloc` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_alloc`，承担模块行为中的一个聚焦逻辑片段。

### Lines 320-326: function alloc with evict / 函数 alloc with evict
```python
def _alloc_with_evict(env, n):
    """Alloc *n* tokens, evicting if necessary.  Returns tensor or None."""
    v = _alloc(env, n)
    if v is None:
        env.tree.evict(EvictParams(num_tokens=n * 2, mamba_num=2))
        v = _alloc(env, n)
    return v
```
**EN:** Alloc *n* tokens, evicting if necessary. This block implements `_alloc_with_evict` and captures one focused piece of the module's behavior.
**CN:** Alloc *n* tokens, evicting if necessary. 该代码块实现 `_alloc_with_evict`，承担模块行为中的一个聚焦逻辑片段。

### Lines 329-340: function insert seq / 函数 insert seq
```python
def _insert_seq(env, seq):
    """Insert one sequence (alloc + evict-fallback).  Returns True on success."""
    v = _alloc_with_evict(env, len(seq))
    if v is None:
        return False
    mamba_val = None
    if env.has_mamba:
        req = env.make_req()
        mamba_val = req.mamba_pool_idx.unsqueeze(0)
    key = RadixKey(seq)
    env.tree.insert(InsertParams(key=key, value=v[: len(key)], mamba_value=mamba_val))
    return True
```
**EN:** Insert one sequence (alloc + evict-fallback). This block implements `_insert_seq` and captures one focused piece of the module's behavior.
**CN:** Insert one sequence (alloc + evict-fallback). 该代码块实现 `_insert_seq`，承担模块行为中的一个聚焦逻辑片段。

### Lines 343-346: function populate / 函数 populate
```python
def _populate(env, count):
    """Insert first *count* sequences (with evict-fallback)."""
    for seq in env.seqs[:count]:
        _insert_seq(env, seq)
```
**EN:** Insert first *count* sequences (with evict-fallback). This block implements `_populate` and captures one focused piece of the module's behavior.
**CN:** Insert first *count* sequences (with evict-fallback). 该代码块实现 `_populate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 349-365: function fill no evict / 函数 fill no evict
```python
def _fill_no_evict(env):
    """Insert sequences until pool exhausted (no eviction).  Returns count."""
    inserted = 0
    for seq in env.seqs:
        v = _alloc(env, len(seq))
        if v is None:
            break
        mamba_val = None
        if env.has_mamba:
            req = env.make_req()
            mamba_val = req.mamba_pool_idx.unsqueeze(0)
        key = RadixKey(seq)
        env.tree.insert(
            InsertParams(key=key, value=v[: len(key)], mamba_value=mamba_val)
        )
        inserted += 1
    return inserted
```
**EN:** Insert sequences until pool exhausted (no eviction). This block implements `_fill_no_evict` and captures one focused piece of the module's behavior.
**CN:** Insert sequences until pool exhausted (no eviction). 该代码块实现 `_fill_no_evict`，承担模块行为中的一个聚焦逻辑片段。

### Lines 366-370: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Benchmark result + runner
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 371-372: class BenchResult declaration / 类 BenchResult 声明
```python
@dataclass
class BenchResult:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 373-377: class-level constants and configuration for `BenchResult` / 类级常量与配置
```python
    name: str
    num_ops: int
    total_tokens: int
    elapsed_s: float
    latencies_us: list[float]
```
**EN:** This block defines shared names such as `name`, `num_ops`, `total_tokens`, `elapsed_s`, `latencies_us`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `name`, `num_ops`, `total_tokens`, `elapsed_s`, `latencies_us` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 379-381: method ops per sec / 方法 ops per sec
```python
    @property
    def ops_per_sec(self):
        return self.num_ops / self.elapsed_s if self.elapsed_s > 0 else 0
```
**EN:** This block implements `ops_per_sec` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `ops_per_sec`，承担模块行为中的一个聚焦逻辑片段。

### Lines 383-385: method tokens per sec / 方法 tokens per sec
```python
    @property
    def tokens_per_sec(self):
        return self.total_tokens / self.elapsed_s if self.elapsed_s > 0 else 0
```
**EN:** This block implements `tokens_per_sec` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `tokens_per_sec`，承担模块行为中的一个聚焦逻辑片段。

### Lines 387-389: method p50 us / 方法 p50 us
```python
    @property
    def p50_us(self):
        return statistics.median(self.latencies_us) if self.latencies_us else 0
```
**EN:** This block implements `p50_us` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `p50_us`，承担模块行为中的一个聚焦逻辑片段。

### Lines 391-396: method p99 us / 方法 p99 us
```python
    @property
    def p99_us(self):
        if not self.latencies_us:
            return 0
        idx = int(len(self.latencies_us) * 0.99)
        return sorted(self.latencies_us)[min(idx, len(self.latencies_us) - 1)]
```
**EN:** This block implements `p99_us` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `p99_us`，承担模块行为中的一个聚焦逻辑片段。

### Lines 398-407: method report / 方法 report
```python
    def report(self):
        tok = (
            f"{self.tokens_per_sec:>12,.0f} tok/s"
            if self.total_tokens > 0
            else f"{'N/A':>12s} tok/s"
        )
        return (
            f"  {self.name:<18s} | {tok} | {self.ops_per_sec:>10,.0f} ops/s | "
            f"p50={self.p50_us:>8,.0f}us  p99={self.p99_us:>8,.0f}us"
        )
```
**EN:** This block implements `report` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `report`，承担模块行为中的一个聚焦逻辑片段。

### Lines 410-451: function bench api / 函数 bench api
```python
def bench_api(
    name, setup_fn, op_fn, num_ops, tokens_per_op=0, warmup=10, verify_fn=None
):
    """Time *op_fn(item)* for each item from *setup_fn()*.

    *verify_fn*, if provided, runs during warmup and once after timing
    (excluded from latency measurement).
    """
    items = setup_fn()
    assert (
        len(items) >= num_ops + warmup
    ), f"need {num_ops + warmup} items, got {len(items)}"

    for i in range(warmup):
        op_fn(items[i])
        if verify_fn:
            verify_fn(items[i])

    gc.collect()
    gc_was = gc.isenabled()
    gc.disable()

    latencies: list[float] = []
    t0 = time.perf_counter()
    for i in range(warmup, warmup + num_ops):
        ts = time.perf_counter()
        op_fn(items[i])
        latencies.append((time.perf_counter() - ts) * 1e6)
    elapsed = time.perf_counter() - t0

    if gc_was:
        gc.enable()
    if verify_fn:
        verify_fn(items[warmup + num_ops - 1])

    return BenchResult(
        name,
        num_ops,
        tokens_per_op * num_ops if tokens_per_op > 0 else 0,
        elapsed,
        latencies,
    )
```
**EN:** Time *op_fn(item)* for each item from *setup_fn()*. This block implements `bench_api` and captures one focused piece of the module's behavior.
**CN:** Time *op_fn(item)* for each item from *setup_fn()*. 该代码块实现 `bench_api`，承担模块行为中的一个聚焦逻辑片段。

### Lines 452-456: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Five benchmark scenarios
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 457-478: function bench insert / 函数 bench insert
```python
def bench_insert(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    tree_cls=None,
    page_size=1,
):
    """Insert throughput (alloc + evict-fallback + insert)."""
    env = _make_env(num_seqs, chunk_len, kv_size, components, tree_cls, page_size)
    warmup = min(20, num_seqs // 10)

    return bench_api(
        "insert",
        lambda: list(range(len(env.seqs))),
        lambda idx: _insert_seq(env, env.seqs[idx]),
        num_seqs - warmup,
        env.avg_tokens,
        warmup,
        (lambda _: env.tree.sanity_check()) if verify else None,
    )
```
**EN:** Insert throughput (alloc + evict-fallback + insert). This block implements `bench_insert` and captures one focused piece of the module's behavior.
**CN:** Insert throughput (alloc + evict-fallback + insert). 该代码块实现 `bench_insert`，承担模块行为中的一个聚焦逻辑片段。

### Lines 481-522: function bench match prefix / 函数 bench match prefix
```python
def bench_match_prefix(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    tree_cls=None,
    page_size=1,
):
    """Prefix matching throughput (hit / partial / miss mix)."""
    env = _make_env(num_seqs, chunk_len, kv_size, components, tree_cls, page_size)
    _populate(env, num_seqs // 2)

    rng = random.Random(123)
    pop = num_seqs // 2
    queries: list[list[int]] = []
    for _ in env.seqs:
        roll = rng.random()
        if roll < 0.33:
            queries.append(env.seqs[rng.randint(0, pop - 1)])
        elif roll < 0.66:
            base = env.seqs[rng.randint(0, pop - 1)]
            queries.append(base + [rng.randint(1, 32000)] * rng.randint(10, 100))
        else:
            queries.append([rng.randint(1, 32000)] * rng.randint(50, 300))

    def verify_fn(q):
        k = RadixKey(q)
        r1 = env.tree.match_prefix(MatchPrefixParams(key=k))
        r2 = env.tree.match_prefix(MatchPrefixParams(key=k))
        assert len(r1.device_indices) == len(r2.device_indices), "match not idempotent"

    warmup = min(20, len(queries) // 10)
    return bench_api(
        "match_prefix",
        lambda: queries,
        lambda q: env.tree.match_prefix(MatchPrefixParams(key=RadixKey(q))),
        min(len(queries) - warmup, num_seqs),
        env.avg_tokens,
        warmup,
        verify_fn if verify else None,
    )
```
**EN:** Prefix matching throughput (hit / partial / miss mix). This block implements `bench_match_prefix` and captures one focused piece of the module's behavior.
**CN:** Prefix matching throughput (hit / partial / miss mix). 该代码块实现 `bench_match_prefix`，承担模块行为中的一个聚焦逻辑片段。

### Lines 525-551: function bench evict / 函数 bench evict
```python
def bench_evict(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    tree_cls=None,
    page_size=1,
):
    """Eviction throughput — fill pool then repeatedly evict batches."""
    env = _make_env(num_seqs, chunk_len, kv_size, components, tree_cls, page_size)
    inserted = _fill_no_evict(env)

    evict_batch = max(100, kv_size // 200)
    num_evictions = max(inserted // 5, 100)
    items = [(evict_batch,)] * (num_evictions + 50)
    warmup = min(20, num_evictions // 10)

    return bench_api(
        "evict",
        lambda: items,
        lambda item: env.tree.evict(EvictParams(num_tokens=item[0], mamba_num=2)),
        num_evictions - warmup,
        evict_batch,
        warmup,
        (lambda _: env.tree.sanity_check()) if verify else None,
    )
```
**EN:** Eviction throughput — fill pool then repeatedly evict batches. This block implements `bench_evict` and captures one focused piece of the module's behavior.
**CN:** Eviction throughput — fill pool then repeatedly evict batches. 该代码块实现 `bench_evict`，承担模块行为中的一个聚焦逻辑片段。

### Lines 554-595: function bench lock unlock / 函数 bench lock unlock
```python
def bench_lock_unlock(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    tree_cls=None,
    page_size=1,
):
    """Lock/unlock throughput — match nodes then cycle lock/unlock."""
    env = _make_env(num_seqs, chunk_len, kv_size, components, tree_cls, page_size)
    _populate(env, num_seqs // 2)

    nodes = []
    for seq in env.seqs[: num_seqs // 2]:
        r = env.tree.match_prefix(MatchPrefixParams(key=RadixKey(seq)))
        if r.last_device_node != env.tree.root_node:
            nodes.append(r.last_device_node)
    if not nodes:
        return BenchResult("lock_unlock", 0, 0, 0, [])

    rng = random.Random(99)
    num_pairs = min(len(nodes) * 2, num_seqs)
    items = [rng.choice(nodes) for _ in range(num_pairs + 50)]

    def op_fn(node):
        lr = env.tree.inc_lock_ref(node)
        env.tree.dec_lock_ref(
            node,
            DecLockRefParams(swa_uuid_for_lock=getattr(lr, "swa_uuid_for_lock", None)),
        )

    warmup = min(20, num_pairs // 10)
    return bench_api(
        "lock_unlock",
        lambda: items,
        op_fn,
        num_pairs - warmup,
        0,
        warmup,
        (lambda _: env.tree.sanity_check()) if verify else None,
    )
```
**EN:** Lock/unlock throughput — match nodes then cycle lock/unlock. This block implements `bench_lock_unlock` and captures one focused piece of the module's behavior.
**CN:** Lock/unlock throughput — match nodes then cycle lock/unlock. 该代码块实现 `bench_lock_unlock`，承担模块行为中的一个聚焦逻辑片段。

### Lines 598-663: function bench cache finished / 函数 bench cache finished
```python
def bench_cache_finished(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    tree_cls=None,
    page_size=1,
):
    """cache_finished_req throughput — full request lifecycle.

    Simulates: match_prefix → inc_lock_ref → alloc → fill req_to_token → cache_finished_req.
    """
    env = _make_env(num_seqs, chunk_len, kv_size, components, tree_cls, page_size)

    # Pre-build Req objects with token IDs filled into req_to_token
    req_items: list = []
    for seq in env.seqs:
        key = RadixKey(seq)
        mr = env.tree.match_prefix(MatchPrefixParams(key=key))
        matched_len = len(mr.device_indices)
        node = mr.last_device_node
        lr = env.tree.inc_lock_ref(node)

        remaining = len(seq) - matched_len
        if remaining > 0:
            v = _alloc_with_evict(env, remaining)
            if v is None:
                env.tree.dec_lock_ref(
                    node,
                    DecLockRefParams(
                        swa_uuid_for_lock=getattr(lr, "swa_uuid_for_lock", None)
                    ),
                )
                continue
            kv_indices = torch.cat([mr.device_indices, v])
        else:
            kv_indices = mr.device_indices

        req = env.make_req()
        req.origin_input_ids = list(seq)
        req.output_ids = []
        req.fill_ids = list(seq)
        req.last_node = node
        req.cache_protected_len = matched_len
        req.kv_committed_len = len(seq)
        req.kv_committed_freed = False
        if hasattr(lr, "swa_uuid_for_lock"):
            req.swa_uuid_for_lock = lr.swa_uuid_for_lock
        env.rtp.req_to_token[req.req_pool_idx, : len(kv_indices)] = kv_indices
        req_items.append(req)

    if not req_items:
        return BenchResult("cache_finished", 0, 0, 0, [])

    warmup = min(20, len(req_items) // 10)
    return bench_api(
        "cache_finished",
        lambda: req_items,
        lambda req: env.tree.cache_finished_req(req, is_insert=True),
        len(req_items) - warmup,
        env.avg_tokens,
        warmup,
        # Pool math doesn't hold here (many reqs still hold allocated tokens).
        (lambda _: env.tree.sanity_check()) if verify else None,
    )
```
**EN:** cache_finished_req throughput — full request lifecycle. This block implements `bench_cache_finished` and captures one focused piece of the module's behavior.
**CN:** cache_finished_req throughput — full request lifecycle. 该代码块实现 `bench_cache_finished`，承担模块行为中的一个聚焦逻辑片段。

### Lines 664-668: supporting source context / 辅助源码上下文
```python


# ===================================================================
# Runner
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 669-675: module-level constants and configuration / 模块级常量与配置
```python
ALL_BENCHMARKS = {
    "insert": bench_insert,
    "match": bench_match_prefix,
    "evict": bench_evict,
    "lock": bench_lock_unlock,
    "cache_finished": bench_cache_finished,
}
```
**EN:** This block defines shared names such as `ALL_BENCHMARKS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `ALL_BENCHMARKS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 678-725: helper routine run all benchmarks / 辅助流程 run all benchmarks
```python
def run_all_benchmarks(
    num_seqs=5000,
    chunk_len=256,
    kv_size=500_000,
    components=None,
    verify=False,
    benchmarks=None,
    tree_cls=None,
    page_size=1,
):
    if components is None:
        components = _DEFAULT_COMPONENTS
    if benchmarks is None or "all" in benchmarks:
        benchmarks = list(ALL_BENCHMARKS.keys())

    set_global_server_args_for_scheduler(
        ServerArgs(model_path="dummy", page_size=page_size)
    )

    impl_name = (tree_cls or UnifiedRadixCache).__name__
    results = []
    for name in benchmarks:
        if name not in ALL_BENCHMARKS:
            print(f"[WARN] Unknown benchmark: {name}, skipping")
            continue
        results.append(
            ALL_BENCHMARKS[name](
                num_seqs=num_seqs,
                chunk_len=chunk_len,
                kv_size=kv_size,
                components=components,
                verify=verify,
                tree_cls=tree_cls,
                page_size=page_size,
            )
        )

    print("=" * 100)
    print(
        f"{impl_name} Benchmark | "
        f"num_seqs={num_seqs}  chunk_len={chunk_len}  kv_size={kv_size}  "
        f"page_size={page_size}  components={[c.value for c in components]}  verify={verify}"
    )
    print("-" * 100)
    for r in results:
        print(r.report())
    print("=" * 100)
    return results
```
**EN:** This helper encapsulates `run_all_benchmarks` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_all_benchmarks`，以便周围测试复用准备、执行或校验逻辑。

### Lines 726-730: supporting source context / 辅助源码上下文
```python


# ===================================================================
# pytest wrapper
# ===================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 731-774: module-level constants and configuration / 模块级常量与配置
```python
_CI_BENCH_CONFIGS = [
    dict(
        label="FULL_MAMBA_ps1",
        components=(ComponentType.FULL, ComponentType.MAMBA),
        page_size=1,
        num_seqs=5000,
        kv_size=500_000,
    ),
    dict(
        label="FULL_SWA_ps1",
        components=(ComponentType.FULL, ComponentType.SWA),
        page_size=1,
        num_seqs=1000,
        kv_size=100_000,
    ),
    dict(
        label="FULL_ps16",
        components=(ComponentType.FULL,),
        page_size=16,
        num_seqs=1000,
        kv_size=100_000,
    ),
    dict(
        label="FULL_SWA_ps16",
        components=(ComponentType.FULL, ComponentType.SWA),
        page_size=16,
        num_seqs=1000,
        kv_size=100_000,
    ),
    dict(
        label="FULL_ps128",
        components=(ComponentType.FULL,),
        page_size=128,
        num_seqs=1000,
        kv_size=200_000,
    ),
    dict(
        label="FULL_SWA_ps128",
        components=(ComponentType.FULL, ComponentType.SWA),
        page_size=128,
        num_seqs=1000,
        kv_size=200_000,
    ),
]
```
**EN:** This block defines shared names such as `_CI_BENCH_CONFIGS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_CI_BENCH_CONFIGS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 777-777: class _BenchSuite declaration / 类 _BenchSuite 声明
```python
class _BenchSuite:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 778-778: supporting statements / 辅助语句
```python
    """Mixin: subclass must set bench_cfg dict with keys: label, components, page_size, num_seqs, kv_size."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 780-784: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        set_global_server_args_for_scheduler(
            ServerArgs(model_path="dummy", page_size=cls.bench_cfg["page_size"])
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 786-797: method run / 方法 run
```python
    def _run(self, bench_fn):
        cfg = self.bench_cfg
        r = bench_fn(
            cfg["num_seqs"],
            _BENCH_CHUNK_LEN,
            cfg["kv_size"],
            components=cfg["components"],
            verify=True,
            page_size=cfg["page_size"],
        )
        self.assertGreater(r.num_ops, 0)
        self.assertGreater(r.ops_per_sec, 0)
```
**EN:** This block implements `_run` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。

### Lines 799-800: test case bench insert / 测试用例 bench insert
```python
    def test_bench_insert(self):
        self._run(bench_insert)
```
**EN:** This test exercises `test_bench_insert` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_insert`。

### Lines 802-803: test case bench match prefix / 测试用例 bench match prefix
```python
    def test_bench_match_prefix(self):
        self._run(bench_match_prefix)
```
**EN:** This test exercises `test_bench_match_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_match_prefix`。

### Lines 805-806: test case bench evict / 测试用例 bench evict
```python
    def test_bench_evict(self):
        self._run(bench_evict)
```
**EN:** This test exercises `test_bench_evict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_evict`。

### Lines 808-809: test case bench lock unlock / 测试用例 bench lock unlock
```python
    def test_bench_lock_unlock(self):
        self._run(bench_lock_unlock)
```
**EN:** This test exercises `test_bench_lock_unlock` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_lock_unlock`。

### Lines 811-812: test case bench cache finished / 测试用例 bench cache finished
```python
    def test_bench_cache_finished(self):
        self._run(bench_cache_finished)
```
**EN:** This test exercises `test_bench_cache_finished` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bench_cache_finished`。

### Lines 815-873: module-level constants and configuration / 模块级常量与配置
```python
for _cfg in _CI_BENCH_CONFIGS:
    _name = f"TestBench_{_cfg['label']}"
    globals()[_name] = type(
        _name,
        (_BenchSuite, unittest.TestCase),
        {"bench_cfg": _cfg},
    )
    globals()[_name].__module__ = __name__
del _cfg, _name


# ===================================================================
# CLI
# ===================================================================
_TREE_CONFIGS = {
    "full": ((ComponentType.FULL,), None),
    "mamba": ((ComponentType.FULL, ComponentType.MAMBA), None),
    "swa": ((ComponentType.FULL, ComponentType.SWA), None),
    "all": ((ComponentType.FULL, ComponentType.SWA, ComponentType.MAMBA), None),
    "legacy-mamba": ((ComponentType.FULL, ComponentType.MAMBA), MambaRadixCache),
    "legacy-swa": ((ComponentType.FULL, ComponentType.SWA), SWARadixCache),
}

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="UnifiedRadixCache benchmark")
    parser.add_argument("--num-seqs", type=int, default=5000)
    parser.add_argument("--chunk-len", type=int, default=256)
    parser.add_argument("--kv-size", type=int, default=500_000)
    parser.add_argument(
        "--components",
        nargs="+",
        choices=list(_TREE_CONFIGS.keys()),
        default=["mamba", "legacy-mamba"],
        help="Component configs to benchmark",
    )
    parser.add_argument("--page-size", type=int, default=1)
    parser.add_argument(
        "--verify", action="store_true", help="Enable correctness assertions"
    )
    parser.add_argument(
        "--benchmarks",
        nargs="+",
        default=["all"],
        help="insert match evict lock cache_finished all",
    )
    args, _ = parser.parse_known_args()

    for comp_name in args.components:
        components, tree_cls = _TREE_CONFIGS[comp_name]
        run_all_benchmarks(
            num_seqs=args.num_seqs,
            chunk_len=args.chunk_len,
            kv_size=args.kv_size,
            components=components,
            verify=args.verify,
            benchmarks=args.benchmarks,
            tree_cls=tree_cls,
            page_size=args.page_size,
        )
```
**EN:** This block defines shared names such as `_name`, `_TREE_CONFIGS`, `parser`, `args`, `_`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_name`, `_TREE_CONFIGS`, `parser`, `args`, `_` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

## Key Concepts / 关键概念
- `_suppress_logs`: This block implements `_suppress_logs` and captures one focused piece of the module's behavior. / 该代码块实现 `_suppress_logs`，承担模块行为中的一个聚焦逻辑片段。
- `_full_attention_layer_ids`: This block implements `_full_attention_layer_ids` and captures one focused piece of the module's behavior. / 该代码块实现 `_full_attention_layer_ids`，承担模块行为中的一个聚焦逻辑片段。
- `_non_full_layer_ids`: This block implements `_non_full_layer_ids` and captures one focused piece of the module's behavior. / 该代码块实现 `_non_full_layer_ids`，承担模块行为中的一个聚焦逻辑片段。
- `gen_random_sequences`: Generate *num_seqs* token sequences with tree-like prefix sharing. / 该代码块实现 `gen_random_sequences`，承担模块行为中的一个聚焦逻辑片段。
- `create_bench_cache`: Create cache. / 该辅助函数封装了 `create_bench_cache`，以便周围测试复用准备、执行或校验逻辑。
- `_Env`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_make_env`: Create sequences + cache, return shared _Env. / 该代码块实现 `_make_env`，承担模块行为中的一个聚焦逻辑片段。
- `_alloc`: This block implements `_alloc` and captures one focused piece of the module's behavior. / 该代码块实现 `_alloc`，承担模块行为中的一个聚焦逻辑片段。
- `BenchResult.ops_per_sec`: This block implements `ops_per_sec` and captures one focused piece of the module's behavior. / 该代码块实现 `ops_per_sec`，承担模块行为中的一个聚焦逻辑片段。
- `BenchResult.tokens_per_sec`: This block implements `tokens_per_sec` and captures one focused piece of the module's behavior. / 该代码块实现 `tokens_per_sec`，承担模块行为中的一个聚焦逻辑片段。
- `BenchResult.p50_us`: This block implements `p50_us` and captures one focused piece of the module's behavior. / 该代码块实现 `p50_us`，承担模块行为中的一个聚焦逻辑片段。
- `BenchResult.p99_us`: This block implements `p99_us` and captures one focused piece of the module's behavior. / 该代码块实现 `p99_us`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `gc`, `logging`, `random`, `statistics`, `time`, `unittest`, `contextlib`, `dataclasses`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.configs.mamba_utils`, `sglang.srt.environ`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.cache_init_params`, `sglang.srt.mem_cache.mamba_radix_cache`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.mem_cache.swa_radix_cache`, `sglang.srt.mem_cache.unified_cache_components.tree_component`, `sglang.srt.mem_cache.unified_radix_cache`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 873
