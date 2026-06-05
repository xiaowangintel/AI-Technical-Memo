# invariant_checker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/invariant_checker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements invariant checker logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 invariant checker 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-12: Provide supporting module logic / 提供辅助模块逻辑
```python
import logging
import warnings
from dataclasses import dataclass
from typing import (
    TYPE_CHECKING,
    Callable,
    List,
    Optional,
    Tuple,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 14-28: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.environ import envs
from sglang.srt.managers.scheduler_components.pool_stats_observer import (
    PoolStats,
    SchedulerPoolStatsObserver,
)
from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils.common import (
    ceil_align,
    raise_error_or_warn,
)
from sglang.srt.utils.watchdog import WatchdogRaw
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 30-31: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.scheduler import Scheduler
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 34-56: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(kw_only=True, slots=True)
class SchedulerInvariantChecker:
    is_hybrid_swa: bool
    is_hybrid_ssm: bool
    disaggregation_mode: DisaggregationMode
    page_size: int
    full_tokens_per_layer: Optional[int]
    swa_tokens_per_layer: Optional[int]
    max_total_num_tokens: int
    server_args: ServerArgs
    tree_cache: BasePrefixCache
    token_to_kv_pool_allocator: BaseTokenToKVPoolAllocator
    req_to_token_pool: ReqToTokenPool
    pool_stats_observer: SchedulerPoolStatsObserver
    get_last_batch: Callable
    get_running_batch: Callable
    count_req_pool_leak_warnings: int = 0
    count_memory_leak_warnings: int = 0

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 57-73: Implement check pool invariant / 实现检查 池 invariant
```python
    def _check_pool_invariant(
        pool_name: str,
        available: int,
        evictable: int,
        protected: int,
        session_held: int,
        total: int,
        uncached: int = 0,
    ) -> Tuple[bool, str]:
        """Check: available + evictable + protected + session_held + uncached == total."""
        total_accounted = available + evictable + protected + session_held + uncached
        leak = total_accounted != total
        msg = (
            f"[{pool_name}] {total=}, {available=}, {evictable=}, "
            f"{protected=}, {session_held=}, {uncached=}"
        )
        return leak, msg
```
**EN:** This block implements the method `_check_pool_invariant(pool_name, available, evictable, protected, session_held, ...)` on `SchedulerInvariantChecker`. It focuses on Check: available + evictable + protected + session_held + uncached == total., so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_pool_invariant(pool_name, available, evictable, protected, session_held, ...)`。它围绕 `_check_pool_invariant` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 75-96: Implement check full pool / 实现检查 full 池
```python
    def _check_full_pool(self, ps: PoolStats, uncached: int = 0) -> Tuple[bool, str]:
        if self.is_hybrid_swa:
            protected = self.tree_cache.full_protected_size()
            session_held = self.pool_stats_observer.session_held_full_tokens()
            total = self.full_tokens_per_layer
        elif self.is_hybrid_ssm and self.tree_cache.supports_mamba():
            protected = self.tree_cache.full_protected_size()
            session_held = self.pool_stats_observer.session_held_tokens()
            total = self.token_to_kv_pool_allocator.size
        else:
            protected = self.tree_cache.protected_size()
            session_held = self.pool_stats_observer.session_held_tokens()
            total = self.max_total_num_tokens
        return self._check_pool_invariant(
            "full",
            ps.full_available_size,
            ps.full_evictable_size,
            protected,
            session_held,
            total,
            uncached,
        )
```
**EN:** This block implements the method `_check_full_pool(ps, uncached)` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_check_full_pool`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_full_pool(ps, uncached)`。它围绕 `_check_full_pool` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 98-107: Implement check swa pool / 实现检查 swa 池
```python
    def _check_swa_pool(self, ps: PoolStats, uncached: int = 0) -> Tuple[bool, str]:
        return self._check_pool_invariant(
            "swa",
            ps.swa_available_size,
            ps.swa_evictable_size,
            self.tree_cache.swa_protected_size(),
            self.pool_stats_observer.session_held_swa_tokens(),
            self.swa_tokens_per_layer,
            uncached,
        )
```
**EN:** This block implements the method `_check_swa_pool(ps, uncached)` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_check_swa_pool`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_swa_pool(ps, uncached)`。它围绕 `_check_swa_pool` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 109-145: Implement check mamba pool / 实现检查 mamba 池
```python
    def _check_mamba_pool(self, ps: PoolStats) -> Tuple[bool, str]:
        leak, msg = self._check_pool_invariant(
            "mamba",
            ps.mamba_available_size,
            ps.mamba_evictable_size,
            self.tree_cache.mamba_protected_size(),
            self.pool_stats_observer.session_held_mamba_slots(),
            self.req_to_token_pool.mamba_pool.size,
        )
        if leak:
            # Page-level leak diagnosis for mamba
            free_full_pages = set(
                self.token_to_kv_pool_allocator.free_pages.tolist()
                + self.token_to_kv_pool_allocator.release_pages.tolist()
            )
            cached_full_pages = set(self.tree_cache.all_values_flatten().tolist())
            expected_full_pages = set(
                range(1, self.token_to_kv_pool_allocator.size + 1)
            )
            leaked_full_pages = (
                expected_full_pages - free_full_pages - cached_full_pages
            )
            free_mamba_pages = set(
                self.req_to_token_pool.mamba_pool.free_slots.tolist()
            )
            cached_mamba_pages = set(
                self.tree_cache.all_mamba_values_flatten().tolist()
            )
            expected_mamba_pages = set(range(self.req_to_token_pool.mamba_pool.size))
            leaked_mamba_pages = (
                expected_mamba_pages - free_mamba_pages - cached_mamba_pages
            )
            msg += (
                f", leaked_full_pages={leaked_full_pages or None}"
                f", leaked_mamba_pages={leaked_mamba_pages or None}"
            )
        return leak, msg
```
**EN:** This block implements the method `_check_mamba_pool(ps)` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_check_mamba_pool`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_mamba_pool(ps)`。它围绕 `_check_mamba_pool` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 147-185: Implement get total uncached sizes / 实现get total uncached sizes
```python
    def _get_total_uncached_sizes(
        self,
    ) -> Tuple[int, int]:
        """Sum uncached tokens for full and SWA pools across all active batches.

        Returns (full_uncached, swa_uncached). For non-SWA models, swa_uncached is 0.

        For full pool: uncached = allocated - cache_protected_len
        For SWA pool:  uncached = allocated - max(cache_protected_len, swa_evicted_seqlen)
        """
        # After decode: running_batch IS last_batch (same object), count once.
        # After prefill: they differ, both hold uncached tokens.
        batches = [self.get_last_batch()]
        if (
            self.get_running_batch() not in (None, self.get_last_batch())
            and not self.get_running_batch().is_empty()
        ):
            batches.append(self.get_running_batch())

        full_uncached = 0
        swa_uncached = 0
        for batch in batches:
            for req in batch.reqs:
                assert req.kv_committed_freed == req.kv_overallocated_freed
                if req.kv_committed_freed or req.req_pool_idx is None:
                    continue

                allocated_len = req.kv_allocated_len
                if self.page_size > 1:
                    allocated_len = ceil_align(allocated_len, self.page_size)
                    assert req.cache_protected_len % self.page_size == 0

                full_uncached += allocated_len - req.cache_protected_len
                if self.is_hybrid_swa:
                    swa_uncached += allocated_len - max(
                        req.cache_protected_len, req.swa_evicted_seqlen
                    )

        return full_uncached, swa_uncached
```
**EN:** This block implements the method `_get_total_uncached_sizes()` on `SchedulerInvariantChecker`. It focuses on Sum uncached tokens for full and SWA pools across all active batches., so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_get_total_uncached_sizes()`。它围绕 `_get_total_uncached_sizes` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 187-212: Implement self check during busy / 实现self 检查 during busy
```python
    def self_check_during_busy(self):
        if self.get_last_batch() is None:
            return

        spec_topk = self.server_args.speculative_eagle_topk or 1
        if spec_topk > 1:
            warnings.warn(
                "Runtime memory check (busy) is not supported when speculation topk > 1."
            )
            return

        ps = self.pool_stats_observer.get_pool_stats()
        full_uncached, swa_uncached = self._get_total_uncached_sizes()

        full_leak, full_msg = self._check_full_pool(ps, uncached=full_uncached)

        swa_leak, swa_msg = False, ""
        if self.is_hybrid_swa:
            swa_leak, swa_msg = self._check_swa_pool(ps, uncached=swa_uncached)

        if envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.get() > 1:
            logger.info(f"[Mem Check (BUSY)] {full_msg}")
            if swa_msg:
                logger.info(f"[Mem Check (BUSY)] {swa_msg}")
        assert not full_leak, f"Full Pool Mem Leak Detected! {full_msg}"
        assert not swa_leak, f"SWA Pool Mem Leak Detected! {swa_msg}"
```
**EN:** This block implements the method `self_check_during_busy()` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `self_check_during_busy`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `self_check_during_busy()`。它围绕 `self_check_during_busy` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 214-235: Implement check req pool / 实现检查 req 池
```python
    def _check_req_pool(self):
        if self.disaggregation_mode == DisaggregationMode.DECODE:
            req_total_size = (
                self.req_to_token_pool.size + self.req_to_token_pool.pre_alloc_size
            )
        else:
            req_total_size = self.req_to_token_pool.size

        session_req_count = self.pool_stats_observer.session_held_req_count()
        if len(self.req_to_token_pool.free_slots) + session_req_count != req_total_size:
            msg = (
                "req_to_token_pool memory leak detected!"
                f"available_size={len(self.req_to_token_pool.free_slots)}, "
                f"session_held={session_req_count}, "
                f"total_size={self.req_to_token_pool.size}\n"
            )
            raise_error_or_warn(
                self,
                envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_IDLE.get(),
                "count_req_pool_leak_warnings",
                msg,
            )
```
**EN:** This block implements the method `_check_req_pool()` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_check_req_pool`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_req_pool()`。它围绕 `_check_req_pool` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 237-244: Implement report leak / 实现report leak
```python
    def _report_leak(self, pool_name: str, token_msg: str):
        msg = f"{pool_name} memory leak detected! {token_msg}"
        raise_error_or_warn(
            self,
            envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_IDLE.get(),
            "count_memory_leak_warnings",
            msg,
        )
```
**EN:** This block implements the method `_report_leak(pool_name, token_msg)` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_report_leak`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_report_leak(pool_name, token_msg)`。它围绕 `_report_leak` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 246-267: Implement check all pools / 实现检查 all pools
```python
    def _check_all_pools(
        self, ps: PoolStats, uncached: int = 0
    ) -> Tuple[bool, List[str]]:
        """Check memory invariant across all pools. Returns (has_leak, messages)."""
        has_leak = False
        messages = []

        full_leak, full_msg = self._check_full_pool(ps, uncached=uncached)
        has_leak |= full_leak
        messages.append(full_msg)

        if self.is_hybrid_swa:
            swa_leak, swa_msg = self._check_swa_pool(ps)
            has_leak |= swa_leak
            messages.append(swa_msg)

        if self.is_hybrid_ssm and self.tree_cache.supports_mamba():
            mamba_leak, mamba_msg = self._check_mamba_pool(ps)
            has_leak |= mamba_leak
            messages.append(mamba_msg)

        return has_leak, messages
```
**EN:** This block implements the method `_check_all_pools(ps, uncached)` on `SchedulerInvariantChecker`. It focuses on Check memory invariant across all pools., so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_all_pools(ps, uncached)`。它围绕 `_check_all_pools` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 269-275: Implement check tree cache / 实现检查 tree 缓存
```python
    def _check_tree_cache(self):
        if (
            self.tree_cache.is_tree_cache()
            and (self.is_hybrid_swa and self.tree_cache.supports_swa())
            or (self.is_hybrid_ssm and self.tree_cache.supports_mamba())
        ):
            self.tree_cache.sanity_check()
```
**EN:** This block implements the method `_check_tree_cache()` on `SchedulerInvariantChecker`. It focuses on handling the invariant checker responsibilities represented by `_check_tree_cache`, so the class can advance the invariant checker workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerInvariantChecker` 上的方法 `_check_tree_cache()`。它围绕 `_check_tree_cache` 所承担的 invariant checker 相关职责展开，使该类能够独立推进相应流程。

### Lines 278-299: Implement create scheduler watchdog / 实现create 调度器 watchdog
```python
def create_scheduler_watchdog(
    scheduler: "Scheduler", watchdog_timeout: float, soft: bool = False
) -> WatchdogRaw:
    def dump_info() -> str:
        if scheduler.is_initializing:
            return ""
        _, messages = scheduler.invariant_checker._check_all_pools(
            scheduler.pool_stats_observer.get_pool_stats(),
        )
        return (
            f"{scheduler.cur_batch.batch_size()=}\n"
            f"{scheduler.cur_batch.reqs=}\n" + "\n".join(messages)
        )

    return WatchdogRaw(
        debug_name="Scheduler",
        get_counter=lambda: scheduler.forward_ct,
        is_active=lambda: scheduler.is_initializing or scheduler.cur_batch is not None,
        watchdog_timeout=watchdog_timeout,
        soft=soft,
        dump_info=dump_info,
    )
```
**EN:** This block implements the function `create_scheduler_watchdog(scheduler, watchdog_timeout, soft)`. It focuses on handling the invariant checker responsibilities represented by `create_scheduler_watchdog`, providing reusable behavior for the invariant checker pipeline.
**CN:** 该代码块实现函数 `create_scheduler_watchdog(scheduler, watchdog_timeout, soft)`。它围绕 `create_scheduler_watchdog` 所承担的 invariant checker 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerInvariantChecker
- **Main callables / 主要可调用对象**: create_scheduler_watchdog
- **Domain focus / 领域焦点**: invariant checker / invariant checker
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, typing, warnings
- **Third-party / 第三方库**: __future__
- **Local Modules / 本地模块**: sglang.srt.disaggregation.utils, sglang.srt.environ, sglang.srt.managers.scheduler, sglang.srt.managers.scheduler_components.pool_stats_observer, sglang.srt.mem_cache.allocator, sglang.srt.mem_cache.base_prefix_cache, sglang.srt.mem_cache.memory_pool, sglang.srt.server_args, sglang.srt.utils.common, sglang.srt.utils.watchdog
