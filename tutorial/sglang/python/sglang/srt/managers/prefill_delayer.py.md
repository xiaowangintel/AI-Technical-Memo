# prefill_delayer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/prefill_delayer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements prefill delayer logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 prefill delayer 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5: Import runtime dependencies / 导入运行时依赖
```python
import dataclasses
import logging
import time
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, NamedTuple, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 7-7: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-10: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.environ import envs
from sglang.srt.utils import get_bool_env_var
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 12-13: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.observability.metrics_collector import SchedulerMetricsCollector
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 15-15: Provide supporting module logic / 提供辅助模块逻辑
```python
_DEBUG_LOG = get_bool_env_var("SGLANG_PREFILL_DELAYER_DEBUG_LOG")
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-20: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(frozen=True)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-26: Define class _State / 定义类 _State
```python
class _State:
    delayed_count: int = 0
    start_time: float = field(default_factory=time.perf_counter)

    def bump_delayed_count(self) -> "_State":
        return dataclasses.replace(self, delayed_count=self.delayed_count + 1)
```
**EN:** This block declares the class `_State`. It centers on coordinating prefill delayer behavior, with methods such as bump_delayed_count.
**CN:** 该代码块声明类 `_State`。它负责承载与 prefill delayer 相关的核心状态与行为，并通过 bump_delayed_count 等方法组织实现。

### Lines 29-35: Define class _NegotiateOutput / 定义类 _NegotiateOutput
```python
class _NegotiateOutput(NamedTuple):
    next_state: Optional[_State]
    input_estimation: str
    output_allow: bool
    output_reason: str
    num_prefillable: int
    num_token_watermark_force_allow: int
```
**EN:** This block declares the class `_NegotiateOutput`. It centers on coordinating prefill delayer behavior.
**CN:** 该代码块声明类 `_NegotiateOutput`。它负责承载与 prefill delayer 相关的核心状态与行为。

### Lines 38-38: Provide supporting module logic / 提供辅助模块逻辑
```python
class PrefillDelayer:
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 39-74: Initialize PrefillDelayer / 初始化 PrefillDelayer
```python
    def __init__(
        self,
        dp_size: int,
        attn_tp_size: int,
        cpu_group,
        server_args,
        max_delay_passes: int,
        token_usage_low_watermark: Optional[float],
        metrics_collector: Optional["SchedulerMetricsCollector"] = None,
        device: Optional["torch.device"] = "cpu",
        device_group=None,
    ):
        self._max_delay_passes = max_delay_passes
        self._token_usage_low_watermark = token_usage_low_watermark
        # Queue-based trigger is opt-in: activates only when queue_min_ratio
        # is explicitly set. Additive with the slot-based trigger.
        self._queue_min_ratio = server_args.prefill_delayer_queue_min_ratio
        # Fall back to 5000ms if unset; this is a local safety cap, not a
        # semantic default, so we don't surface it via ServerArgs.
        self._max_delay_ms = server_args.prefill_delayer_max_delay_ms
        if self._max_delay_ms is None:
            self._max_delay_ms = 5000.0
        self._queue_trigger_enabled = self._queue_min_ratio is not None
        logger.info(
            f"PrefillDelayer initialized with "
            f"max_delay_passes={self._max_delay_passes} "
            f"token_usage_low_watermark={self._token_usage_low_watermark} "
            f"queue_min_ratio={self._queue_min_ratio} "
            f"max_delay_ms={self._max_delay_ms} "
            f"queue_trigger_enabled={self._queue_trigger_enabled}"
        )
        self.dp_size = dp_size
        self.enable_dp_attention = server_args.enable_dp_attention
        dp_size_dim = dp_size if self.enable_dp_attention else 1

        # Mirror scheduler_dp_attn_mixin's NCCL all-gather path: when the
```
**EN:** This block implements the initializer `__init__(dp_size, attn_tp_size, cpu_group, server_args, max_delay_passes, ...)` for `PrefillDelayer`. It prepares the object state and connects the instance to the surrounding prefill delayer workflow.
**CN:** 该代码块实现 `PrefillDelayer` 的初始化方法 `__init__(dp_size, attn_tp_size, cpu_group, server_args, max_delay_passes, ...)`。它负责准备对象状态，并把实例接入 prefill delayer 相关的运行流程。

### Lines 75-107: Initialize PrefillDelayer (continued) / 初始化 PrefillDelayer（续）
```python
        # env flag is on (or overlap scheduling is disabled), ride the NCCL
        # device group on `device` instead of gloo on CPU.
        use_nccl = (
            server_args.disable_overlap_schedule
            or envs.SGLANG_NCCL_ALL_GATHER_IN_OVERLAP_SCHEDULER_SYNC_BATCH.get()
        )
        if use_nccl:
            assert (
                device_group is not None
            ), "device_group is required when using NCCL for PrefillDelayer all-gather"
            self._gather_group = device_group
            self._gather_device = device
        else:
            self._gather_group = cpu_group
            self._gather_device = "cpu"

        # Fields packed per rank into the all-gather tensor: prefillable,
        # token_watermark_force_allow, running_batch, max_prefill_bs,
        # waiting_queue_len.
        self._global_info_buffer = torch.empty(
            (dp_size_dim, attn_tp_size, 5),
            dtype=torch.int64,
            device=self._gather_device,
        )

        self._metrics_collector = metrics_collector

        self._curr_state: Optional[_State] = None
        self.skip_first_delayer = True

        assert (
            not server_args.disable_overlap_schedule
        ), "To use PrefillDelayer, disable_overlap_schedule must be False."
```
**EN:** This block implements the initializer `__init__(dp_size, attn_tp_size, cpu_group, server_args, max_delay_passes, ...)` for `PrefillDelayer`. It prepares the object state and connects the instance to the surrounding prefill delayer workflow.
**CN:** 该代码块实现 `PrefillDelayer` 的初始化方法 `__init__(dp_size, attn_tp_size, cpu_group, server_args, max_delay_passes, ...)`。它负责准备对象状态，并把实例接入 prefill delayer 相关的运行流程。

### Lines 109-128: Implement negotiate should allow prefill / 实现negotiate should allow prefill
```python
    def _negotiate_should_allow_prefill(
        self,
        local_prefillable: bool,
        token_usage: float,
        running_batch: int = 0,
        max_prefill_bs: int = 0,
        max_running_requests: int = 0,
        waiting_queue_len: int = 0,
    ) -> _NegotiateOutput:
        out = self._negotiate_should_allow_prefill_pure(
            prev_state=self._curr_state,
            local_prefillable=local_prefillable,
            token_usage=token_usage,
            running_batch=running_batch,
            max_prefill_bs=max_prefill_bs,
            max_running_requests=max_running_requests,
            waiting_queue_len=waiting_queue_len,
        )
        self._curr_state = out.next_state
        return out
```
**EN:** This block implements the method `_negotiate_should_allow_prefill(local_prefillable, token_usage, running_batch, max_prefill_bs, max_running_requests, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill(local_prefillable, token_usage, running_batch, max_prefill_bs, max_running_requests, ...)`。它围绕 `_negotiate_should_allow_prefill` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 129-130: Import runtime dependencies / 导入运行时依赖
```python

    # (Almost) pure function, do not modify self state
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 131-166: Implement negotiate should allow prefill pure / 实现negotiate should allow prefill pure
```python
    def _negotiate_should_allow_prefill_pure(
        self,
        prev_state: Optional[_State],
        local_prefillable: bool,
        token_usage: float,
        running_batch: int = 0,
        max_prefill_bs: int = 0,
        max_running_requests: int = 0,
        waiting_queue_len: int = 0,
    ) -> _NegotiateOutput:
        # Compute local states
        local_token_watermark_force_allow = (
            local_prefillable
            and ((x := self._token_usage_low_watermark) is not None)
            and (token_usage < x)
        )

        # Gather global states
        tp0_info = self._gather_info(
            local_prefillable=local_prefillable,
            local_token_watermark_force_allow=local_token_watermark_force_allow,
            running_batch=running_batch,
            max_prefill_bs=max_prefill_bs,
            waiting_queue_len=waiting_queue_len,
        )
        global_prefillable = tp0_info[:, 0]
        global_token_watermark_force_allow = tp0_info[:, 1]
        global_running_batch = tp0_info[:, 2]
        global_max_prefill_bs = tp0_info[:, 3]
        global_waiting_queue_len = tp0_info[:, 4]

        # Compute derived global states
        if global_prefillable.min().item() > 0:
            prefillable_status = "all"
        elif global_prefillable.max().item() == 0:
            prefillable_status = "none"
```
**EN:** This block implements the method `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill_pure`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)`。它围绕 `_negotiate_should_allow_prefill_pure` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 167-202: Continue negotiate should allow prefill pure / 继续说明negotiate should allow prefill pure
```python
        else:
            prefillable_status = "mixed"
        global_exists_token_watermark_force_allow = (
            global_token_watermark_force_allow.max().item() > 0
        )
        debug_info = dict(
            input_estimation=prefillable_status,
            num_prefillable=global_prefillable.sum().item(),
            num_token_watermark_force_allow=global_token_watermark_force_allow.sum().item(),
        )

        # Compute outputs
        if prefillable_status == "all":
            # Safety valve: low KV usage means GPU is underutilized, skip
            # delay. Mirrors the check in the "mixed" branch.
            if global_exists_token_watermark_force_allow:
                return _NegotiateOutput(
                    next_state=None,
                    output_allow=True,
                    output_reason="token_watermark",
                    **debug_info,
                )

            if not self.enable_dp_attention:
                max_running_requests = (
                    max_running_requests + self.dp_size - 1
                ) // self.dp_size

            global_running_batch_max = int(global_running_batch.max().item())
            global_max_prefill_bs_max = int(global_max_prefill_bs.max().item())
            global_waiting_queue_max = int(global_waiting_queue_len.max().item())

            # Queue-based trigger: delay prefill until the waiting queue
            # reaches queue_min = min(running_req * ratio, max_prefill_bs),
            # capped by a wall-clock timeout to bound worst-case TTFT.
            # Targets workloads where decode requests finish one-at-a-time
```
**EN:** This block implements the method `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill_pure`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)`。它围绕 `_negotiate_should_allow_prefill_pure` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 203-238: Continue negotiate should allow prefill pure / 继续说明negotiate should allow prefill pure
```python
            # and fragment prefill into many tiny batches.
            queue_condition = False
            if self._queue_trigger_enabled and global_running_batch_max > 0:
                queue_min_effective = min(
                    int(global_running_batch_max * self._queue_min_ratio),
                    global_max_prefill_bs_max,
                )
                queue_condition = (
                    queue_min_effective > 0
                    and global_waiting_queue_max < queue_min_effective
                )
                if queue_condition and prev_state is not None:
                    elapsed_ms = (time.perf_counter() - prev_state.start_time) * 1000.0
                    if elapsed_ms >= self._max_delay_ms:
                        queue_condition = False

            slot_condition = (
                max_running_requests - global_running_batch_max
                < global_max_prefill_bs_max
            )

            if slot_condition or queue_condition:
                # When the "max_decode_bs - running_bs < max_prefill_bs" condition is met,
                # the first merge_batch causes the decoding to fail to reach the maximum batch size.
                if self.skip_first_delayer:
                    self.skip_first_delayer = False
                    pass
                else:
                    next_state = prev_state or _State()
                    next_state = next_state.bump_delayed_count()
                    return _NegotiateOutput(
                        next_state=next_state,
                        output_allow=False,
                        output_reason="delay",
                        **debug_info,
                    )
```
**EN:** This block implements the method `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill_pure`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)`。它围绕 `_negotiate_should_allow_prefill_pure` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 239-274: Continue negotiate should allow prefill pure / 继续说明negotiate should allow prefill pure
```python
            exist_previous_wait = prev_state is not None
            return _NegotiateOutput(
                next_state=None,
                output_allow=True,
                output_reason="wait_success" if exist_previous_wait else "no_wait",
                **debug_info,
            )
        elif prefillable_status == "none":
            return _NegotiateOutput(
                next_state=None,
                # It does not matter whether we allow or not, thus we allow for simplicity
                output_allow=True,
                output_reason="",
                **debug_info,
            )
        elif prefillable_status == "mixed":
            if global_exists_token_watermark_force_allow:
                return _NegotiateOutput(
                    next_state=None,
                    output_allow=True,
                    output_reason="token_watermark",
                    **debug_info,
                )

            prev_delayed_count = prev_state.delayed_count if prev_state else 0
            if prev_delayed_count < self._max_delay_passes - 1:
                next_state = prev_state or _State()
                next_state = next_state.bump_delayed_count()
                return _NegotiateOutput(
                    next_state=next_state,
                    output_allow=False,
                    output_reason="delay",
                    **debug_info,
                )
            else:
                return _NegotiateOutput(
```
**EN:** This block implements the method `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill_pure`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)`。它围绕 `_negotiate_should_allow_prefill_pure` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 275-281: Continue negotiate should allow prefill pure / 继续说明negotiate should allow prefill pure
```python
                    next_state=None,
                    output_allow=True,
                    output_reason="wait_timeout",
                    **debug_info,
                )
        else:
            raise NotImplementedError
```
**EN:** This block implements the method `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_negotiate_should_allow_prefill_pure`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_negotiate_should_allow_prefill_pure(prev_state, local_prefillable, token_usage, running_batch, max_prefill_bs, ...)`。它围绕 `_negotiate_should_allow_prefill_pure` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 283-308: Implement gather info / 实现gather info
```python
    def _gather_info(
        self,
        local_prefillable: bool,
        local_token_watermark_force_allow: bool,
        running_batch: int = 0,
        max_prefill_bs: int = 0,
        waiting_queue_len: int = 0,
    ):
        local_info = torch.tensor(
            [
                int(local_prefillable),
                int(local_token_watermark_force_allow),
                running_batch,
                max_prefill_bs,
                waiting_queue_len,
            ],
            device=self._gather_device,
            dtype=torch.int64,
        )
        torch.distributed.all_gather_into_tensor(
            self._global_info_buffer.flatten(),
            local_info,
            group=self._gather_group,
        )
        tp0_info = self._global_info_buffer[:, 0, :]
        return tp0_info
```
**EN:** This block implements the method `_gather_info(local_prefillable, local_token_watermark_force_allow, running_batch, max_prefill_bs, waiting_queue_len)` on `PrefillDelayer`. It focuses on handling the prefill delayer responsibilities represented by `_gather_info`, so the class can advance the prefill delayer workflow in a self-contained way.
**CN:** 该代码块实现 `PrefillDelayer` 上的方法 `_gather_info(local_prefillable, local_token_watermark_force_allow, running_batch, max_prefill_bs, waiting_queue_len)`。它围绕 `_gather_info` 所承担的 prefill delayer 相关职责展开，使该类能够独立推进相应流程。

### Lines 311-348: Define class PrefillDelayerSinglePassExecutor / 定义类 PrefillDelayerSinglePassExecutor
```python
class PrefillDelayerSinglePassExecutor:
    def __init__(self, prefill_delayer: PrefillDelayer, token_usage: float):
        self._prefill_delayer = prefill_delayer
        self._token_usage = token_usage
        self._result: Optional[_NegotiateOutput] = None

    @property
    def _called(self) -> bool:
        return self._result is not None

    def finalize(self, *, actual_prefill: bool):
        if not self._called:
            self.negotiate_should_allow_prefill(local_prefillable=False)

        _record_single_pass_result(
            actual_execution=actual_prefill,
            output=self._result,
            metrics_collector=self._prefill_delayer._metrics_collector,
        )

    def negotiate_should_allow_prefill(
        self,
        local_prefillable: bool,
        running_batch: int = 0,
        max_prefill_bs: int = 0,
        max_running_requests: int = 0,
        waiting_queue_len: int = 0,
    ) -> bool:
        if not self._called:
            self._result = self._prefill_delayer._negotiate_should_allow_prefill(
                local_prefillable=local_prefillable,
                token_usage=self._token_usage,
                running_batch=running_batch,
                max_prefill_bs=max_prefill_bs,
                max_running_requests=max_running_requests,
                waiting_queue_len=waiting_queue_len,
            )
        return self._result.output_allow
```
**EN:** This block declares the class `PrefillDelayerSinglePassExecutor`. It centers on coordinating prefill delayer behavior, with methods such as __init__, _called, finalize, negotiate_should_allow_prefill.
**CN:** 该代码块声明类 `PrefillDelayerSinglePassExecutor`。它负责承载与 prefill delayer 相关的核心状态与行为，并通过 __init__, _called, finalize, negotiate_should_allow_prefill 等方法组织实现。

### Lines 351-391: Implement record single pass result / 实现record single pass result
```python
def _record_single_pass_result(
    actual_execution: bool,
    output: _NegotiateOutput,
    metrics_collector: Optional["SchedulerMetricsCollector"],
) -> None:
    if _DEBUG_LOG:
        if output.output_allow and (output.output_reason == "wait_timeout"):
            logger.info(
                f"PrefillDelayer timeout thus not forbid prefill "
                f"(num_prefillable={output.num_prefillable}, "
                f"actual_execution={actual_execution})"
            )
        elif output.output_allow and (output.output_reason == "token_watermark"):
            logger.info(
                f"PrefillDelayer force allow prefill due to low watermark. "
                f"(num_prefillable={output.num_prefillable}, "
                f"num_token_watermark_force_allow={output.num_token_watermark_force_allow}, "
                f"actual_execution={actual_execution})"
            )
        else:
            assert output.output_reason in {
                "",
                "wait_success",
                "no_wait",
                "delay",
            }

    if metrics_collector is not None:
        if (s := output.next_state) is not None:
            wait_seconds = time.perf_counter() - s.start_time
            forward_passes = s.delayed_count
        else:
            wait_seconds = forward_passes = 0
        metrics_collector.observe_prefill_delayer_outcome(
            forward_passes=forward_passes,
            wait_seconds=wait_seconds,
            input_estimation=output.input_estimation,
            output_allow=output.output_allow,
            output_reason=output.output_reason,
            actual_execution=actual_execution,
        )
```
**EN:** This block implements the function `_record_single_pass_result(actual_execution, output, metrics_collector)`. It focuses on handling the prefill delayer responsibilities represented by `_record_single_pass_result`, providing reusable behavior for the prefill delayer pipeline.
**CN:** 该代码块实现函数 `_record_single_pass_result(actual_execution, output, metrics_collector)`。它围绕 `_record_single_pass_result` 所承担的 prefill delayer 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: _State, _NegotiateOutput, PrefillDelayer, PrefillDelayerSinglePassExecutor
- **Main callables / 主要可调用对象**: _record_single_pass_result
- **Domain focus / 领域焦点**: prefill delayer / prefill delayer
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, time, typing
- **Third-party / 第三方库**: torch
- **Local Modules / 本地模块**: sglang.srt.environ, sglang.srt.observability.metrics_collector, sglang.srt.utils
