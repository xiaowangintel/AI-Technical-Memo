# scheduler_status_logger.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/scheduler_status_logger.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for logging helpers. / 为 SGLang 运行时提供面向日志辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import time
from typing import TYPE_CHECKING, List, Optional

import torch.distributed as dist

from sglang.srt.environ import envs
from sglang.srt.utils.log_utils import create_log_targets, log_json

if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `time`, `typing`, `torch.distributed`, `sglang.srt.environ`, `sglang.srt.utils.log_utils`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `time`, `typing`, `torch.distributed`, `sglang.srt.environ`, `sglang.srt.utils.log_utils`。

### Lines 15-15: Class `SchedulerStatusLogger` declaration / 类 `SchedulerStatusLogger` 声明
```python
class SchedulerStatusLogger:
```
**EN:** This class establishes `SchedulerStatusLogger` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `maybe_create`, `maybe_dump`.
**CN:** 该类将 `SchedulerStatusLogger` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `maybe_create`, `maybe_dump` 等方法。

### Lines 16-20: Method `SchedulerStatusLogger.__init__` / 方法 `SchedulerStatusLogger.__init__`
```python
    def __init__(self, targets: List[str], dump_interval: float):
        self.loggers = create_log_targets(targets=targets, name_prefix=__name__)
        self.dump_interval = dump_interval
        self.last_dump_time = 0.0
        self.rank = dist.get_rank() if dist.is_initialized() else 0
```
**EN:** This method implements `__init__` on `SchedulerStatusLogger`. It primarily calls `create_log_targets`, `dist.is_initialized`, `dist.get_rank` to complete its work. State updates are written into `self.loggers`, `self.dump_interval`, `self.last_dump_time`, `self.rank`.
**CN:** 该方法（属于 `SchedulerStatusLogger`）实现了 `__init__`。 它主要通过调用 `create_log_targets`, `dist.is_initialized`, `dist.get_rank` 来完成任务。 状态更新主要写入 `self.loggers`, `self.dump_interval`, `self.last_dump_time`, `self.rank`。

### Lines 22-37: Method `SchedulerStatusLogger.maybe_create` / 方法 `SchedulerStatusLogger.maybe_create`
```python
    @staticmethod
    def maybe_create(enable_metrics: bool) -> Optional["SchedulerStatusLogger"]:
        target = envs.SGLANG_LOG_SCHEDULER_STATUS_TARGET.get()
        if not target:
            return None

        if not enable_metrics:
            raise ValueError(
                "SGLANG_LOG_SCHEDULER_STATUS_TARGET is set but --enable-metrics "
                "is not active. Status dumps require --enable-metrics to work."
            )

        return SchedulerStatusLogger(
            targets=[t.strip() for t in target.split(",") if t.strip()],
            dump_interval=envs.SGLANG_LOG_SCHEDULER_STATUS_INTERVAL.get(),
        )
```
**EN:** This method implements `maybe_create` on `SchedulerStatusLogger`. It primarily calls `envs.SGLANG_LOG_SCHEDULER_STATUS_TARGET.get`, `SchedulerStatusLogger`, `ValueError`, `envs.SGLANG_LOG_SCHEDULER_STATUS_INTERVAL.get`, `t.strip`, `target.split` to complete its work. State updates are written into `target`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `SchedulerStatusLogger`）实现了 `maybe_create`。 它主要通过调用 `envs.SGLANG_LOG_SCHEDULER_STATUS_TARGET.get`, `SchedulerStatusLogger`, `ValueError`, `envs.SGLANG_LOG_SCHEDULER_STATUS_INTERVAL.get`, `t.strip`, `target.split` 来完成任务。 状态更新主要写入 `target`。 实现中使用了条件分支。

### Lines 39-55: Method `SchedulerStatusLogger.maybe_dump` / 方法 `SchedulerStatusLogger.maybe_dump`
```python
    def maybe_dump(
        self, running_batch: "ScheduleBatch", waiting_queue: List["Req"]
    ) -> None:
        now = time.time()
        if now - self.last_dump_time < self.dump_interval:
            return

        self.last_dump_time = now
        log_json(
            self.loggers,
            "scheduler.status",
            {
                "rank": self.rank,
                "running_rids": [r.rid for r in running_batch.reqs],
                "queued_rids": [r.rid for r in waiting_queue],
            },
        )
```
**EN:** This method implements `maybe_dump` on `SchedulerStatusLogger`. It primarily calls `time.time`, `log_json` to complete its work. State updates are written into `now`, `self.last_dump_time`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `SchedulerStatusLogger`）实现了 `maybe_dump`。 它主要通过调用 `time.time`, `log_json` 来完成任务。 状态更新主要写入 `now`, `self.last_dump_time`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `SchedulerStatusLogger`
- **Functions / 函数**: `__init__`, `maybe_create`, `maybe_dump`
- **Themes / 主题**: `scheduler`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.utils.log_utils`, `sglang.srt.managers.schedule_batch`
- **External / 外部依赖**: `torch.distributed`
- **Standard library / 标准库**: `__future__`, `time`, `typing`
