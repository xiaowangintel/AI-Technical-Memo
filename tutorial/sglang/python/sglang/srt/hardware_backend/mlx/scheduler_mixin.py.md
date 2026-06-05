# scheduler_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/scheduler_mixin.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for scheduler integration inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的调度器集成支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Module setup and shared state / 模块设置与共享状态
```python
"""MLX overlap scheduling mixin for the SGLang scheduler.

Provides ``event_loop_overlap_mlx``, which pipelines MLX forward
passes by keeping two in-flight lazy graphs queued on the GPU while
the scheduler runs its CPU-side bookkeeping on the tokens of the
older one.  The lazy-graph primitives live in
``hardware_backend/mlx/tp_worker.py`` and ``model_runner.py``.

Each request's KV lives ina set of per-request, per-layer ``ContiguousKVCache``
objects that the ``MLXAttentionWrapper`` mutates in place during the forward pass.
Chained decodes reuse the same cache objects: step N+1's graph reads
step N's lazy writes via MLX's dependency tracking, so the GPU runs
both steps back-to-back with no idle gap.
"""

from __future__ import annotations

import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, List, Optional

import mlx.core as mx

from sglang.srt.environ import envs
from sglang.srt.utils import DynamicGradMode

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.hardware_backend.mlx.model_runner import (
        MlxPendingDecode,
        MlxPendingExtend,
        MlxPendingPrefill,
    )
    from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
    from sglang.srt.managers.scheduler import Scheduler
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `logging`, `dataclasses`, `typing`, `mlx.core`, `sglang.srt.environ`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `logging`, `dataclasses`, `typing`, `mlx.core`, `sglang.srt.environ`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 39-77: Class `MlxPendingJob` declaration / 类 `MlxPendingJob` 声明
```python
@dataclass
class MlxPendingJob:
    """Unfinished MLX work and graphs queued on the GPU.

    Attributes:
        lazy_tokens: Lazily evaluated token IDs produced by the forward
            pass.  Unevaluated; calling ``.tolist()`` / ``.item()`` /
            ``mx.eval`` on it will block until the Metal kernel finishes.
            ``None`` for idle batches.
        prefills: MLX prefill state returned by the model worker — one
            entry per new request in an extend batch.  Used by
            ``finalize_mlx_result`` to commit per-request caches.  Empty
            list for pure-decode steps.
        extends: Chunked-prefill-continuation state, one entry per
            already-active request whose extend seq_len > 1.  Also empty
            for pure-decode steps.
        decode: Decode state covering full-decode mode AND mixed
            single-token decodes inside an extend batch.  Used as the
            chaining root by :meth:`async_chained_decode_mlx`.
        mode: One of ``"decode"``, ``"extend"``, ``"idle"`` describing
            which forward pass produced this job.  Drives finalise
            dispatch and whether chaining is safe.
        batch_copy: Snapshot of the :class:`ScheduleBatch` at launch
            time.  Decoupled from the live batch so
            ``process_batch_result`` can update request state without
            racing against the next scheduling decision.
        reqs: Snapshot of ``batch.reqs`` at launch time.  The overlap
            loop uses this to check ``req.finished()`` on the previous
# ... omitted for brevity ...
    prefills: list["MlxPendingPrefill"]
    extends: list["MlxPendingExtend"]
    decode: Optional["MlxPendingDecode"]
    mode: str
    batch_copy: "ScheduleBatch"
    reqs: List[Req]
```
**EN:** This class establishes `MlxPendingJob` as a compact data container for the surrounding logic. The main stored fields include `lazy_tokens`, `prefills`, `extends`, `decode`, `mode`, `batch_copy`.
**CN:** 该类将 `MlxPendingJob` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `lazy_tokens`, `prefills`, `extends`, `decode`, `mode`, `batch_copy`。

### Lines 80-81: Class `SchedulerMlxOverlapMixin` declaration / 类 `SchedulerMlxOverlapMixin` 声明
```python
class SchedulerMlxOverlapMixin:
    """Mixin that adds MLX overlap scheduling to :class:`Scheduler`."""
```
**EN:** This class establishes `SchedulerMlxOverlapMixin` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `event_loop_overlap_mlx`.
**CN:** 该类将 `SchedulerMlxOverlapMixin` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `event_loop_overlap_mlx` 等方法。

### Lines 83-233: Method `SchedulerMlxOverlapMixin.event_loop_overlap_mlx` / 方法 `SchedulerMlxOverlapMixin.event_loop_overlap_mlx`
```python
    @DynamicGradMode()
    def event_loop_overlap_mlx(self: "Scheduler"):
        """MLX-specific overlap loop modelled on ``mlx_lm.generate.generate_step``.

        At steady state we keep TWO in-flight MLX graphs queued on the
        GPU:

        * ``pending_curr`` — the step whose tokens we are about to block
          on and feed into the scheduler's bookkeeping.
        * ``pending_next`` — the step that was built on top of
          ``pending_curr``'s still-lazy output tokens via
          ``async_chained_decode_mlx`` and has already been handed to
          ``mx.async_eval``.  Because MLX tracks the full dependency
          graph, the GPU will execute ``pending_next`` back-to-back
          with ``pending_curr`` — there is no scheduling gap on the
          device.

        Bookkeeping timeline for a steady-state decode loop:

            iter k:
              build pending_next  (CPU graph build + mx.async_eval; cheap)
              block on pending_curr via .tolist() (wait only on curr's tokens)
              process_batch_result(pending_curr)   <-- GPU is running pending_next
              pending_curr = pending_next

        The chain is broken (we fall back to a "schedule + launch" step)
        whenever any of the following holds:

# ... omitted for brevity ...
            else:
                self.on_idle()

            self.last_batch = next_batch
            if envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.get():
                self.invariant_checker.self_check_during_busy()
```
**EN:** This method implements `event_loop_overlap_mlx` on `SchedulerMlxOverlapMixin`. It primarily calls `DynamicGradMode`, `self.tp_worker.finalize_mlx_result`, `self.process_batch_result`, `self.tp_worker.async_forward_batch_generation_mlx`, `MlxPendingJob`, `self.tp_worker.async_chained_decode_mlx` to complete its work. State updates are written into `pending_curr`, `pending_next`, `result`, `recv_reqs`, `can_chain`, `finished_any`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `SchedulerMlxOverlapMixin`）实现了 `event_loop_overlap_mlx`。 它主要通过调用 `DynamicGradMode`, `self.tp_worker.finalize_mlx_result`, `self.process_batch_result`, `self.tp_worker.async_forward_batch_generation_mlx`, `MlxPendingJob`, `self.tp_worker.async_chained_decode_mlx` 来完成任务。 状态更新主要写入 `pending_curr`, `pending_next`, `result`, `recv_reqs`, `can_chain`, `finished_any`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `MlxPendingJob`, `SchedulerMlxOverlapMixin`
- **Functions / 函数**: `event_loop_overlap_mlx`, `_finalize`, `_launch_fresh`, `_launch_chained`
- **Themes / 主题**: `scheduler`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.srt.hardware_backend.mlx.model_runner`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.scheduler`
- **External / 外部依赖**: `mlx.core`
- **Standard library / 标准库**: `__future__`, `logging`, `dataclasses`, `typing`
