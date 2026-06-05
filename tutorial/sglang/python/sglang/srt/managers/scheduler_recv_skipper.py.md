# scheduler_recv_skipper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_recv_skipper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements scheduler recv skipper logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 调度器 recv skipper 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.environ import envs
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.server_args import ServerArgs
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-38: Define class SchedulerRecvSkipper / 定义类 SchedulerRecvSkipper
```python
class SchedulerRecvSkipper:
    @staticmethod
    def maybe_create(server_args: ServerArgs):
        if server_args.scheduler_recv_interval <= 1:
            return None
        return SchedulerRecvSkipper(server_args)

    def __init__(self, server_args: ServerArgs):
        # Can be supported if needed, but may need e.g. `global_forward_mode`
        assert not server_args.enable_dp_attention
        self._counter = 0
        self._threshold = server_args.scheduler_recv_interval
        # All can be tuned if needed
        self._default_weight = envs.SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_DEFAULT.get()
        self._weight_of_forward_mode = {
            ForwardMode.DECODE: envs.SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_DECODE.get(),
            ForwardMode.TARGET_VERIFY: envs.SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_TARGET_VERIFY.get(),
            None: envs.SGLANG_SCHEDULER_RECV_SKIPPER_WEIGHT_NONE.get(),
        }

    def handle(self, last_forward_mode: ForwardMode):
        should_recv = False

        last_weight = self._weight_of_forward_mode.get(
            last_forward_mode, self._default_weight
        )
        self._counter += last_weight

        if self._counter >= self._threshold:
            self._counter = 0
            should_recv = True

        return should_recv
```
**EN:** This block declares the class `SchedulerRecvSkipper`. It centers on coordinating scheduler recv skipper behavior, with methods such as maybe_create, __init__, handle.
**CN:** 该代码块声明类 `SchedulerRecvSkipper`。它负责承载与 调度器 recv skipper 相关的核心状态与行为，并通过 maybe_create, __init__, handle 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerRecvSkipper
- **Domain focus / 领域焦点**: scheduler recv skipper / 调度器 recv skipper
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: None / 无
- **Third-party / 第三方库**: None / 无
- **Local Modules / 本地模块**: sglang.srt.environ, sglang.srt.model_executor.forward_batch_info, sglang.srt.server_args
