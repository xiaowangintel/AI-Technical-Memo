# pool_stats_observer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/pool_stats_observer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements pool stats observer logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 池 stats observer 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-12: Provide supporting module logic / 提供辅助模块逻辑
```python
import dataclasses
from dataclasses import dataclass
from typing import (
    TYPE_CHECKING,
    Any,
    Callable,
    List,
    Optional,
    Tuple,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 14-17: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
    from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
    from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-20: Define class SchedulerStats / 定义类 SchedulerStats
```python
class SchedulerStats: ...  # type: ignore[no-redef]
```
**EN:** This block declares the class `SchedulerStats`. It centers on coordinating pool stats observer behavior.
**CN:** 该代码块声明类 `SchedulerStats`。它负责承载与 池 stats observer 相关的核心状态与行为。

### Lines 21-29: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclasses.dataclass
class PoolStats:
    # For full pools (required)
    full_num_used: int
    full_token_usage: float
    full_available_size: int
    full_evictable_size: int
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 31-51: Provide supporting module logic / 提供辅助模块逻辑
```python
    is_hybrid_swa: bool = False
    is_hybrid_ssm: bool = False
    is_hisparse: bool = False

    # For hybrid-swa pools
    swa_num_used: Optional[int] = None
    swa_token_usage: Optional[float] = None
    swa_available_size: Optional[int] = None
    swa_evictable_size: Optional[int] = None

    # For mamba pools
    mamba_num_used: Optional[int] = None
    mamba_usage: Optional[float] = None
    mamba_available_size: Optional[int] = None
    mamba_evictable_size: Optional[int] = None

    # HiSparse device/host breakdown for decode logs (plain KV pool only)
    hisparse_device_tokens: Optional[int] = None
    hisparse_device_token_usage: Optional[float] = None
    hisparse_host_tokens: Optional[int] = None
    hisparse_host_token_usage: Optional[float] = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 53-62: Implement get kv token stats / 实现get kv Token stats
```python
    def get_kv_token_stats(self) -> Tuple[int, float]:
        # NOTE: mamba pool is not included in the "token usage" calculation.
        if self.is_hybrid_swa:
            num_used = max(self.full_num_used, self.swa_num_used)
            token_usage = max(self.full_token_usage, self.swa_token_usage)
        else:
            num_used = self.full_num_used
            token_usage = self.full_token_usage

        return num_used, token_usage
```
**EN:** This block implements the method `get_kv_token_stats()` on `PoolStats`. It focuses on handling the pool stats observer responsibilities represented by `get_kv_token_stats`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `PoolStats` 上的方法 `get_kv_token_stats()`。它围绕 `get_kv_token_stats` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 64-71: Implement get max pool usage / 实现get max 池 usage
```python
    def get_max_pool_usage(self) -> float:
        usage = self.full_token_usage
        if self.is_hybrid_swa:
            usage = max(usage, self.swa_token_usage)
        if self.is_hybrid_ssm:
            usage = max(usage, self.mamba_usage)
        assert usage is not None and usage >= 0, f"{usage=} is not valid"
        return usage
```
**EN:** This block implements the method `get_max_pool_usage()` on `PoolStats`. It focuses on handling the pool stats observer responsibilities represented by `get_max_pool_usage`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `PoolStats` 上的方法 `get_max_pool_usage()`。它围绕 `get_max_pool_usage` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 73-86: Implement get prefill usage msg parts / 实现get prefill usage msg parts
```python
    def get_prefill_usage_msg_parts(self) -> List[str]:
        parts = []
        if self.is_hybrid_swa:
            parts += [
                f"full token usage: {self.full_token_usage:.2f}",
                f"swa token usage: {self.swa_token_usage:.2f}",
            ]
        if self.is_hybrid_ssm:
            if not self.is_hybrid_swa:
                parts.append(f"full token usage: {self.full_token_usage:.2f}")
            parts.append(f"mamba usage: {self.mamba_usage:.2f}")
        if not parts:
            parts.append(f"token usage: {self.full_token_usage:.2f}")
        return parts
```
**EN:** This block implements the method `get_prefill_usage_msg_parts()` on `PoolStats`. It focuses on handling the pool stats observer responsibilities represented by `get_prefill_usage_msg_parts`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `PoolStats` 上的方法 `get_prefill_usage_msg_parts()`。它围绕 `get_prefill_usage_msg_parts` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 88-118: Implement get decode usage msg parts / 实现get 解码 usage msg parts
```python
    def get_decode_usage_msg_parts(self) -> List[str]:
        parts = []
        if self.is_hybrid_swa:
            parts += [
                f"#full token: {self.full_num_used}",
                f"full token usage: {self.full_token_usage:.2f}",
                f"#swa token: {self.swa_num_used}",
                f"swa token usage: {self.swa_token_usage:.2f}",
            ]
        if self.is_hybrid_ssm:
            if not self.is_hybrid_swa:
                parts += [
                    f"#full token: {self.full_num_used}",
                    f"full token usage: {self.full_token_usage:.2f}",
                ]
            parts += [
                f"mamba num: {self.mamba_num_used}",
                f"mamba usage: {self.mamba_usage:.2f}",
            ]
        if self.is_hisparse:
            parts += [
                f"#gpu token: {self.hisparse_device_tokens}",
                f"gpu token usage: {self.hisparse_device_token_usage:.2f}",
                f"#cpu token: {self.hisparse_host_tokens}",
                f"cpu token usage: {self.hisparse_host_token_usage:.2f}",
            ]
        if not parts:
            parts.append(
                f"#token: {self.full_num_used}, token usage: {self.full_token_usage:.2f}"
            )
        return parts
```
**EN:** This block implements the method `get_decode_usage_msg_parts()` on `PoolStats`. It focuses on handling the pool stats observer responsibilities represented by `get_decode_usage_msg_parts`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `PoolStats` 上的方法 `get_decode_usage_msg_parts()`。它围绕 `get_decode_usage_msg_parts` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 120-138: Implement update scheduler stats / 实现update 调度器 stats
```python
    def update_scheduler_stats(self, stats: SchedulerStats) -> None:
        """Update pool-related fields on SchedulerStats."""
        num_used, _ = self.get_kv_token_stats()
        stats.num_used_tokens = num_used
        stats.token_usage = round(self.get_max_pool_usage(), 2)
        stats.full_token_usage = self.full_token_usage
        if self.is_hybrid_swa:
            stats.swa_token_usage = self.swa_token_usage
            stats.swa_available_tokens = self.swa_available_size
            stats.swa_evictable_tokens = self.swa_evictable_size
            stats.swa_used_tokens = self.swa_num_used
        if self.is_hybrid_ssm:
            stats.mamba_usage = self.mamba_usage
            stats.mamba_available_tokens = self.mamba_available_size
            stats.mamba_evictable_tokens = self.mamba_evictable_size
            stats.mamba_used_tokens = self.mamba_num_used
        stats.kv_available_tokens = self.full_available_size
        stats.kv_evictable_tokens = self.full_evictable_size
        stats.kv_used_tokens = self.full_num_used
```
**EN:** This block implements the method `update_scheduler_stats(stats)` on `PoolStats`. It focuses on Update pool-related fields on SchedulerStats., so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `PoolStats` 上的方法 `update_scheduler_stats(stats)`。它围绕 `update_scheduler_stats` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 139-155: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass(kw_only=True, slots=True, frozen=True)
class SchedulerPoolStatsObserver:
    tree_cache: "BasePrefixCache"
    token_to_kv_pool_allocator: "BaseTokenToKVPoolAllocator"
    req_to_token_pool: "ReqToTokenPool"
    session_controller: Any
    hisparse_coordinator: Any
    is_hybrid_swa: bool
    is_hybrid_ssm: bool
    enable_hisparse: bool
    full_tokens_per_layer: Any
    swa_tokens_per_layer: Any
    max_total_num_tokens: int
    get_last_batch: Callable
    get_running_batch: Callable
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 157-162: Implement streaming session count / 实现streaming 会话 count
```python
    def streaming_session_count(self) -> int:
        return sum(
            1
            for session in self.session_controller.sessions.values()
            if session.streaming
        )
```
**EN:** This block implements the method `streaming_session_count()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `streaming_session_count`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `streaming_session_count()`。它围绕 `streaming_session_count` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 164-177: Implement active pool idxs / 实现active 池 idxs
```python
    def active_pool_idxs(self) -> set:
        """Pool idxs currently owned by reqs in last_batch / running_batch.

        Used to decide which session slots' KV is owned by batch reqs
        (and thus counted via uncached_size, not session_held).
        """
        idxs = set()
        for batch in [self.get_last_batch(), self.get_running_batch()]:
            if batch is None or batch.is_empty():
                continue
            for req in batch.reqs:
                if req.req_pool_idx is not None:
                    idxs.add(req.req_pool_idx)
        return idxs
```
**EN:** This block implements the method `active_pool_idxs()` on `SchedulerPoolStatsObserver`. It focuses on Pool idxs currently owned by reqs in last_batch / running_batch., so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `active_pool_idxs()`。它围绕 `active_pool_idxs` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 179-180: Implement session held tokens / 实现会话 held tokens
```python
    def session_held_tokens(self) -> int:
        return self.tree_cache.session_held_tokens(self.active_pool_idxs())
```
**EN:** This block implements the method `session_held_tokens()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `session_held_tokens`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `session_held_tokens()`。它围绕 `session_held_tokens` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 182-183: Implement session held full tokens / 实现会话 held full tokens
```python
    def session_held_full_tokens(self) -> int:
        return self.tree_cache.session_held_full_tokens(self.active_pool_idxs())
```
**EN:** This block implements the method `session_held_full_tokens()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `session_held_full_tokens`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `session_held_full_tokens()`。它围绕 `session_held_full_tokens` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 185-186: Implement session held swa tokens / 实现会话 held swa tokens
```python
    def session_held_swa_tokens(self) -> int:
        return self.tree_cache.session_held_swa_tokens(self.active_pool_idxs())
```
**EN:** This block implements the method `session_held_swa_tokens()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `session_held_swa_tokens`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `session_held_swa_tokens()`。它围绕 `session_held_swa_tokens` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 188-189: Implement session held req count / 实现会话 held req count
```python
    def session_held_req_count(self) -> int:
        return self.tree_cache.session_held_req_count()
```
**EN:** This block implements the method `session_held_req_count()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `session_held_req_count`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `session_held_req_count()`。它围绕 `session_held_req_count` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 191-192: Implement session held mamba slots / 实现会话 held mamba slots
```python
    def session_held_mamba_slots(self) -> int:
        return self.tree_cache.session_held_mamba_slots(self.active_pool_idxs())
```
**EN:** This block implements the method `session_held_mamba_slots()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `session_held_mamba_slots`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `session_held_mamba_slots()`。它围绕 `session_held_mamba_slots` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 194-214: Implement get pool stats / 实现get 池 stats
```python
    def get_pool_stats(self) -> PoolStats:
        if self.is_hybrid_swa:
            pool_stats = self._get_swa_token_info()
        elif self.is_hybrid_ssm:
            pool_stats = self._get_mamba_token_info()
        else:
            pool_stats = self._get_token_info()

        if self.enable_hisparse:
            pool_stats = self._get_hisparse_token_info(pool_stats)

        # swa + ssm can coexist: overlay mamba fields onto swa stats
        if self.is_hybrid_ssm:
            mamba_stats = self._get_mamba_token_info()
            pool_stats.is_hybrid_ssm = True
            pool_stats.mamba_num_used = mamba_stats.mamba_num_used
            pool_stats.mamba_usage = mamba_stats.mamba_usage
            pool_stats.mamba_available_size = mamba_stats.mamba_available_size
            pool_stats.mamba_evictable_size = mamba_stats.mamba_evictable_size

        return pool_stats
```
**EN:** This block implements the method `get_pool_stats()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `get_pool_stats`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `get_pool_stats()`。它围绕 `get_pool_stats` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 216-226: Implement get token info / 实现get Token info
```python
    def _get_token_info(self) -> PoolStats:
        available_size = self.token_to_kv_pool_allocator.available_size()
        evictable_size = self.tree_cache.evictable_size()
        num_used = self.max_total_num_tokens - (available_size + evictable_size)
        token_usage = num_used / self.max_total_num_tokens
        return PoolStats(
            full_num_used=num_used,
            full_token_usage=token_usage,
            full_available_size=available_size,
            full_evictable_size=evictable_size,
        )
```
**EN:** This block implements the method `_get_token_info()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `_get_token_info`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `_get_token_info()`。它围绕 `_get_token_info` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 228-239: Implement get hisparse token info / 实现get hisparse Token info
```python
    def _get_hisparse_token_info(self, pool_stats: PoolStats) -> PoolStats:
        if self.enable_hisparse and self.hisparse_coordinator is not None:
            h = self.hisparse_coordinator.get_token_stats()
            return dataclasses.replace(
                pool_stats,
                is_hisparse=True,
                hisparse_device_tokens=h.device_tokens,
                hisparse_device_token_usage=h.device_token_usage,
                hisparse_host_tokens=h.host_tokens,
                hisparse_host_token_usage=h.host_token_usage,
            )
        return pool_stats
```
**EN:** This block implements the method `_get_hisparse_token_info(pool_stats)` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `_get_hisparse_token_info`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `_get_hisparse_token_info(pool_stats)`。它围绕 `_get_hisparse_token_info` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 241-272: Implement get mamba token info / 实现get mamba Token info
```python
    def _get_mamba_token_info(self):
        is_mamba_radix_cache = (
            self.tree_cache.supports_mamba() and self.tree_cache.is_tree_cache()
        )
        full_available_size = self.token_to_kv_pool_allocator.available_size()
        full_evictable_size = (
            self.tree_cache.full_evictable_size() if is_mamba_radix_cache else 0
        )
        mamba_available_size = self.req_to_token_pool.mamba_pool.available_size()
        mamba_evictable_size = (
            self.tree_cache.mamba_evictable_size() if is_mamba_radix_cache else 0
        )
        full_num_used = self.token_to_kv_pool_allocator.size - (
            full_available_size + full_evictable_size
        )
        mamba_num_used = self.req_to_token_pool.mamba_pool.size - (
            mamba_available_size + mamba_evictable_size
        )
        full_token_usage = full_num_used / self.token_to_kv_pool_allocator.size
        mamba_usage = mamba_num_used / self.req_to_token_pool.mamba_pool.size

        return PoolStats(
            is_hybrid_ssm=True,
            full_num_used=full_num_used,
            full_token_usage=full_token_usage,
            full_available_size=full_available_size,
            full_evictable_size=full_evictable_size,
            mamba_num_used=mamba_num_used,
            mamba_usage=mamba_usage,
            mamba_available_size=mamba_available_size,
            mamba_evictable_size=mamba_evictable_size,
        )
```
**EN:** This block implements the method `_get_mamba_token_info()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `_get_mamba_token_info`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `_get_mamba_token_info()`。它围绕 `_get_mamba_token_info` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

### Lines 274-305: Implement get swa token info / 实现get swa Token info
```python
    def _get_swa_token_info(self) -> PoolStats:
        full_available_size = self.token_to_kv_pool_allocator.full_available_size()
        full_evictable_size = self.tree_cache.full_evictable_size()
        swa_available_size = self.token_to_kv_pool_allocator.swa_available_size()
        swa_evictable_size = self.tree_cache.swa_evictable_size()
        full_num_used = self.full_tokens_per_layer - (
            full_available_size + full_evictable_size
        )
        swa_num_used = self.swa_tokens_per_layer - (
            swa_available_size + swa_evictable_size
        )
        # FIXME(hisparse): host-backup transiently over-releases the device pool
        # counter, producing negative full_num_used / swa_num_used. We clamp to 0
        # to keep token_usage / leak checks sane, but the underlying accounting
        # bug should be fixed so the clamp can go away.
        if self.enable_hisparse:
            full_num_used = max(0, full_num_used)
            swa_num_used = max(0, swa_num_used)
        full_token_usage = full_num_used / self.full_tokens_per_layer
        swa_token_usage = swa_num_used / self.swa_tokens_per_layer

        return PoolStats(
            is_hybrid_swa=True,
            full_num_used=full_num_used,
            full_token_usage=full_token_usage,
            full_available_size=full_available_size,
            full_evictable_size=full_evictable_size,
            swa_num_used=swa_num_used,
            swa_token_usage=swa_token_usage,
            swa_available_size=swa_available_size,
            swa_evictable_size=swa_evictable_size,
        )
```
**EN:** This block implements the method `_get_swa_token_info()` on `SchedulerPoolStatsObserver`. It focuses on handling the pool stats observer responsibilities represented by `_get_swa_token_info`, so the class can advance the pool stats observer workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerPoolStatsObserver` 上的方法 `_get_swa_token_info()`。它围绕 `_get_swa_token_info` 所承担的 池 stats observer 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerStats, PoolStats, SchedulerPoolStatsObserver
- **Domain focus / 领域焦点**: pool stats observer / 池 stats observer
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, typing
- **Third-party / 第三方库**: __future__
- **Local Modules / 本地模块**: sglang.srt.mem_cache.allocator, sglang.srt.mem_cache.base_prefix_cache, sglang.srt.mem_cache.memory_pool
