# req_time_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/req_time_stats.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `req_time_stats`. The module docstring frames it as: "Utilities for Request Time Stats." / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `req_time_stats` 的逻辑。 它对外提供的主要入口包括 `calibrate_time_diff`, `convert_time_to_realtime`, `convert_time_to_realtime_ns`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-51: Module imports, constants, and setup
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Utilities for Request Time Stats."""

from __future__ import annotations

import logging
import time
import uuid
from dataclasses import dataclass, field
from typing import TYPE_CHECKING, Any, Dict, List, Optional, Union

from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.observability.metrics_collector import (
    SchedulerMetricsCollector,
    TokenizerMetricsCollector,
)
from sglang.srt.observability.trace import (
    SpanAttributes,
    TraceNullContext,
    TraceReqContext,
    TraceSliceContext,
    get_global_tracing_enabled,
)
from sglang.srt.utils import get_bool_env_var

if TYPE_CHECKING:
    from sglang.srt.disaggregation.base.conn import KVTransferMetric
    from sglang.srt.managers.schedule_batch import ScheduleBatch

SGLANG_TEST_REQUEST_TIME_STATS = get_bool_env_var("SGLANG_TEST_REQUEST_TIME_STATS")


logger = logging.getLogger(__name__)

# Reduce system time calls by computing time.time() based on calibrated perf_counter() values.
global_diff_realtime_monotonic = time.time() - time.perf_counter()


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；记录指标或追踪信号。

### Lines 52-56: Function calibrate_time_diff
```python
def calibrate_time_diff():
    # due to NTP, the diff between time.time() and time.perf_counter() can change
    # periodically calibrate the diff
    global global_diff_realtime_monotonic
    global_diff_realtime_monotonic = time.time() - time.perf_counter()
```
**EN:** This callable implements `calibrate_time_diff` and mainly implements calibrate time diff.
**CN:** 这一可调用对象实现了 `calibrate_time_diff`，主要用于实现 calibrate time diff 相关逻辑。

### Lines 57-62: Module-level constants and helpers
```python


real_time = time.time
monotonic_time = time.perf_counter


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 63-66: Function convert_time_to_realtime
```python
def convert_time_to_realtime(time_value: float) -> float:
    # note: Within the time scale of a single request's latency,
    # we assume that the diff does not change significantly.
    return time_value + global_diff_realtime_monotonic
```
**EN:** This callable implements `convert_time_to_realtime`. It takes `time_value` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `convert_time_to_realtime`。它接收 `time_value`，主要用于将数据转换为另一种表示。

### Lines 69-70: Function convert_time_to_realtime_ns
```python
def convert_time_to_realtime_ns(time_value: float) -> int:
    return int((time_value + global_diff_realtime_monotonic) * 1e9)
```
**EN:** This callable implements `convert_time_to_realtime_ns`. It takes `time_value` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `convert_time_to_realtime_ns`。它接收 `time_value`，主要用于将数据转换为另一种表示。

### Lines 73-77: Function convert_time_cross_thread
```python
def convert_time_cross_thread(
    time_value: float, old_diff: float, new_diff: float
) -> float:
    # note: precision loss
    return time_value + old_diff - new_diff
```
**EN:** This callable implements `convert_time_cross_thread`. It takes `time_value`, `old_diff`, `new_diff` and mainly implements convert time cross thread.
**CN:** 这一可调用对象实现了 `convert_time_cross_thread`。它接收 `time_value`, `old_diff`, `new_diff`，主要用于实现 convert time cross thread 相关逻辑。

### Lines 80-95: Class RequestStageConfig
```python
@dataclass
class RequestStageConfig:
    """Configuration for a request pipeline stage.

    Attributes:
        stage_name: Name used for metrics labels and trace span names.
        level: Trace hierarchy depth.
            1 = leaf stages (atomic operations, e.g. TOKENIZE, PREFILL_FORWARD),
            2 = parent/dispatch stages (e.g. API_SERVER_DISPATCH, REQUEST_PROCESS),
            3 = composite/nested stages (e.g. DECODE_LOOP, PREFILL_CHUNKED_FORWARD).
        metrics_is_observed: Whether to call metrics_collector.observe_per_stage_req_latency.
    """

    stage_name: str
    level: int = 0
    metrics_is_observed: bool = False
```
**EN:** This range introduces `RequestStageConfig` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Configuration for a request pipeline stage." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RequestStageConfig`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 98-157: Class RequestStage (part 1/2)
```python
class RequestStage:
    # Tokenizer/gRPC Server
    TOKENIZE = RequestStageConfig(
        "tokenize",
        level=1,
    )
    API_SERVER_DISPATCH = RequestStageConfig(
        "api_server_dispatch",
        level=2,
    )

    # DP controller
    DPC_DISPATCH = RequestStageConfig(
        "dpc_dispatch",
        level=2,
    )

    # common/non-disaggregation
    REQUEST_PROCESS = RequestStageConfig(
        "request_process",
        level=2,
        metrics_is_observed=True,
    )
    PREFILL_WAITING = RequestStageConfig(
        "prefill_waiting",
        level=1,
        # equal to "observe_queue_time"
        metrics_is_observed=False,
    )
    DECODE_FORWARD = RequestStageConfig(
        "decode_forward",
        level=1,
    )
    DECODE_LOOP = RequestStageConfig(
        "decode_loop",
        level=3,
    )
    PREFILL_FORWARD = RequestStageConfig(
        "prefill_forward",
        level=1,
        metrics_is_observed=True,
    )
    PREFILL_CHUNKED_FORWARD = RequestStageConfig(
        "chunked_prefill",
        level=3,
        metrics_is_observed=True,
    )

    # disaggregation prefill
    PREFILL_PREPARE = RequestStageConfig(
        "prefill_prepare",
        level=1,
    )
    PREFILL_BOOTSTRAP = RequestStageConfig(
        "prefill_bootstrap",
        level=1,
        metrics_is_observed=True,
    )
    PREFILL_TRANSFER_KV_CACHE = RequestStageConfig(
        "prefill_transfer_kv_cache",
```
**EN:** This range introduces `RequestStage` and defines the structure or metadata that its methods rely on. This chunk is part 1 of 2 for the same logical block. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RequestStage`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会记录指标或追踪信号。

### Lines 158-217: Class RequestStage (part 2/2)
```python
        level=1,
        metrics_is_observed=True,
    )

    # disaggregation decode
    DECODE_PREPARE = RequestStageConfig(
        "decode_prepare",
        level=1,
        metrics_is_observed=True,
    )
    DECODE_BOOTSTRAP = RequestStageConfig(
        "decode_bootstrap",
        level=1,
        metrics_is_observed=True,
    )
    DECODE_WAITING = RequestStageConfig(
        "decode_waiting",
        level=1,
        metrics_is_observed=True,
    )
    DECODE_TRANSFERRED = RequestStageConfig(
        "decode_transferred",
        level=1,
        metrics_is_observed=True,
    )
    DECODE_FAKE_OUTPUT = RequestStageConfig(
        "fake_output",
        level=3,
        metrics_is_observed=True,
    )
    DECODE_QUICK_FINISH = RequestStageConfig(
        "quick_finish",
        level=1,
        metrics_is_observed=True,
    )

    # speculative decode
    SPEC_DRAFT = RequestStageConfig(
        "spec_draft",
        level=2,
    )

    SPEC_VERIFY = RequestStageConfig(
        "spec_verify",
        level=2,
    )

    SPEC_DRAFT_EXTEND = RequestStageConfig(
        "spec_draft_extend",
        level=3,
    )

    # CPU-side run batch
    RUN_BATCH_CPU = RequestStageConfig(
        "run_batch_cpu",
        level=4,
    )

    # other
    ANONYMOUS = RequestStageConfig("")
```
**EN:** This range introduces `RequestStage` and defines the structure or metadata that its methods rely on. This chunk is part 2 of 2 for the same logical block. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `RequestStage`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会记录指标或追踪信号。

### Lines 220-231: Class ReqTimeStatsBase
```python
@dataclass
class ReqTimeStatsBase:
    enable_metrics: bool = False
    metrics_collector: Optional[
        Union[SchedulerMetricsCollector, TokenizerMetricsCollector]
    ] = None
    trace_ctx: Union[TraceReqContext, TraceNullContext] = field(
        default_factory=TraceNullContext
    )
    disagg_mode: DisaggregationMode = DisaggregationMode.NULL
    diff_realtime_monotonic: float = 0.0

```
**EN:** This range introduces `ReqTimeStatsBase` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `ReqTimeStatsBase`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 232-245: Method ReqTimeStatsBase.new_from_obj
```python
    @classmethod
    def new_from_obj(cls, obj: ReqTimeStatsBase, *args, **kwargs) -> "ReqTimeStatsBase":
        calibrate_time_diff()
        new_obj = cls(*args, **kwargs)
        if obj is None:
            return new_obj
        for key, value in obj.__dict__.items():
            if hasattr(new_obj, key):
                setattr(new_obj, key, value)

        if new_obj.trace_ctx.tracing_enable:
            new_obj.trace_ctx.rebuild_thread_context()

        return new_obj
```
**EN:** This callable implements `ReqTimeStatsBase.new_from_obj`. It takes `cls`, `obj`, `*args`, `**kwargs` and mainly constructs data from an external representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.new_from_obj`。它接收 `cls`, `obj`, `*args`, `**kwargs`，主要用于从外部表示构造数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 247-255: Method ReqTimeStatsBase.disagg_mode_str
```python
    def disagg_mode_str(self) -> str:
        if self.disagg_mode == DisaggregationMode.NULL:
            return "unified"
        elif self.disagg_mode == DisaggregationMode.DECODE:
            return "decode"
        elif self.disagg_mode == DisaggregationMode.PREFILL:
            return "prefill"
        else:
            return "unknown"
```
**EN:** This callable implements `ReqTimeStatsBase.disagg_mode_str` and mainly implements disagg mode str.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.disagg_mode_str`，主要用于实现 disagg mode str 相关逻辑。

### Lines 257-262: Method ReqTimeStatsBase.set_metrics_collector
```python
    def set_metrics_collector(
        self, collector: Union[SchedulerMetricsCollector, TokenizerMetricsCollector]
    ):
        if collector:
            self.enable_metrics = True
            self.metrics_collector = collector
```
**EN:** This callable implements `ReqTimeStatsBase.set_metrics_collector`. It takes `collector` and mainly converts data into another representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.set_metrics_collector`。它接收 `collector`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 264-268: Method ReqTimeStatsBase.observe_per_stage_req_latency
```python
    def observe_per_stage_req_latency(self, stage: RequestStageConfig, latency: float):
        if self.enable_metrics and stage.metrics_is_observed:
            self.metrics_collector.observe_per_stage_req_latency(
                stage.stage_name, latency
            )
```
**EN:** This callable implements `ReqTimeStatsBase.observe_per_stage_req_latency`. It takes `stage`, `latency` and mainly implements observe per stage req latency. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.observe_per_stage_req_latency`。它接收 `stage`, `latency`，主要用于实现 observe per stage req latency 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 270-285: Method ReqTimeStatsBase.init_trace_ctx
```python
    def init_trace_ctx(
        self,
        rid: str,
        bootstrap_room: Optional[int],
        external_trace_header: Optional[Dict[str, str]] = None,
    ):
        self.trace_ctx = TraceReqContext(
            rid=rid,
            bootstrap_room=bootstrap_room,
            role=self.disagg_mode_str(),
            module_name="request",
            external_trace_header=external_trace_header,
        )

        if not self.trace_ctx.tracing_enable:
            self.trace_ctx = TraceNullContext()
```
**EN:** This callable implements `ReqTimeStatsBase.init_trace_ctx`. It takes `rid`, `bootstrap_room`, `external_trace_header` and mainly implements init trace ctx. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.init_trace_ctx`。它接收 `rid`, `bootstrap_room`, `external_trace_header`，主要用于实现 init trace ctx 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 287-302: Method ReqTimeStatsBase.trace_slice
```python
    def trace_slice(
        self,
        stage: RequestStageConfig,
        start_time: float,
        end_time: float,
        attrs: Optional[Dict] = None,
    ):
        if self.trace_ctx.tracing_enable:
            _slice = TraceSliceContext(
                slice_name=stage.stage_name,
                start_time_ns=convert_time_to_realtime_ns(start_time),
                end_time_ns=convert_time_to_realtime_ns(end_time),
                level=stage.level,
                attrs=attrs,
            )
            self.trace_ctx.trace_slice(_slice)
```
**EN:** This callable implements `ReqTimeStatsBase.trace_slice`. It takes `stage`, `start_time`, `end_time`, `attrs` and mainly implements trace slice. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.trace_slice`。它接收 `stage`, `start_time`, `end_time`, `attrs`，主要用于实现 trace slice 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 304-312: Method ReqTimeStatsBase.__getstate__
```python
    def __getstate__(self) -> object:
        # The object is propagated to other processes via serialization and deserialization methods,
        # requiring the metric collector to be reconfigured.
        return {
            "disagg_mode": self.disagg_mode,
            "enable_metrics": False,
            "trace_ctx": self.trace_ctx,
            "diff_realtime_monotonic": global_diff_realtime_monotonic,
        }
```
**EN:** This callable implements `ReqTimeStatsBase.__getstate__` and mainly retrieves a value or derived view. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.__getstate__`，主要用于获取某个值或派生视图。 在这一范围内，它会记录指标或追踪信号。

### Lines 314-322: Method ReqTimeStatsBase.__setstate__
```python
    def __setstate__(self, state: object):
        for key in state.keys():
            if key.endswith("time"):
                state[key] = convert_time_cross_thread(
                    state[key],
                    state["diff_realtime_monotonic"],
                    global_diff_realtime_monotonic,
                )
        self.__dict__.update(state)
```
**EN:** This callable implements `ReqTimeStatsBase.__setstate__`. It takes `state` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ReqTimeStatsBase.__setstate__`。它接收 `state`，主要用于将配置写入可变状态。

### Lines 325-336: Class APIServerReqTimeStats
```python
@dataclass
class APIServerReqTimeStats(ReqTimeStatsBase):
    # get by time.perf_counter()
    created_time: float = 0.0
    finished_time: float = 0.0
    first_token_time: float = 0.0
    last_time: float = 0.0
    tokenize_finish_time: float = 0.0
    api_server_dispatch_time: float = 0.0
    api_server_dispatch_finish_time: float = 0.0
    response_sent_to_client_time: float = 0.0

```
**EN:** This range introduces `APIServerReqTimeStats` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `APIServerReqTimeStats`，并定义其后续方法依赖的结构或元数据。

### Lines 337-346: Method APIServerReqTimeStats.__getstate__
```python
    def __getstate__(self) -> object:
        state = {}
        # send to DP controller or Scheduler
        # If necessary, can propagate the timestamp here, for example:
        # state = {
        #    "created_time": self.created_time,
        #    "api_server_dispatch_time": self.api_server_dispatch_time,
        # }
        state.update(super().__getstate__())
        return state
```
**EN:** This callable implements `APIServerReqTimeStats.__getstate__` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.__getstate__`，主要用于获取某个值或派生视图。

### Lines 348-353: Method APIServerReqTimeStats.set_created_time
```python
    def set_created_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.created_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_req_start(convert_time_to_realtime_ns(ts))
```
**EN:** This callable implements `APIServerReqTimeStats.set_created_time`. It takes `ts` and mainly constructs new objects or contexts. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_created_time`。它接收 `ts`，主要用于构造新的对象或上下文。 在这一范围内，它会记录指标或追踪信号。

### Lines 355-360: Method APIServerReqTimeStats.set_finished_time
```python
    def set_finished_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.finished_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_req_finish(convert_time_to_realtime_ns(ts))
```
**EN:** This callable implements `APIServerReqTimeStats.set_finished_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_finished_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 362-365: Method APIServerReqTimeStats.set_first_token_time
```python
    def set_first_token_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.first_token_time = ts
        self.last_time = ts
```
**EN:** This callable implements `APIServerReqTimeStats.set_first_token_time`. It takes `ts` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_first_token_time`。它接收 `ts`，主要用于将数据转换为另一种表示。

### Lines 367-369: Method APIServerReqTimeStats.set_last_time
```python
    def set_last_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.last_time = ts
```
**EN:** This callable implements `APIServerReqTimeStats.set_last_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_last_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 371-376: Method APIServerReqTimeStats.set_tokenize_finish_time
```python
    def set_tokenize_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.tokenize_finish_time = ts

        stage = RequestStage.TOKENIZE
        self.trace_slice(stage, self.created_time, ts)
```
**EN:** This callable implements `APIServerReqTimeStats.set_tokenize_finish_time`. It takes `ts` and mainly converts data into another representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_tokenize_finish_time`。它接收 `ts`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 378-387: Method APIServerReqTimeStats.set_api_server_dispatch_time
```python
    def set_api_server_dispatch_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.api_server_dispatch_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_slice_start(
                RequestStage.API_SERVER_DISPATCH.stage_name,
                RequestStage.API_SERVER_DISPATCH.level,
                convert_time_to_realtime_ns(ts),
            )
```
**EN:** This callable implements `APIServerReqTimeStats.set_api_server_dispatch_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_api_server_dispatch_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 389-399: Method APIServerReqTimeStats.set_api_server_dispatch_finish_time
```python
    def set_api_server_dispatch_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.api_server_dispatch_finish_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_slice_end(
                RequestStage.API_SERVER_DISPATCH.stage_name,
                RequestStage.API_SERVER_DISPATCH.level,
                convert_time_to_realtime_ns(ts),
                thread_finish_flag=True,
            )
```
**EN:** This callable implements `APIServerReqTimeStats.set_api_server_dispatch_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_api_server_dispatch_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 401-403: Method APIServerReqTimeStats.set_response_sent_to_client_time
```python
    def set_response_sent_to_client_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.response_sent_to_client_time = ts
```
**EN:** This callable implements `APIServerReqTimeStats.set_response_sent_to_client_time`. It takes `ts` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.set_response_sent_to_client_time`。它接收 `ts`，主要用于将数据转换为另一种表示。

### Lines 405-406: Method APIServerReqTimeStats.get_interval
```python
    def get_interval(self):
        return time.perf_counter() - self.last_time
```
**EN:** This callable implements `APIServerReqTimeStats.get_interval` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.get_interval`，主要用于获取某个值或派生视图。

### Lines 408-409: Method APIServerReqTimeStats.get_first_token_latency
```python
    def get_first_token_latency(self):
        return self.first_token_time - self.created_time
```
**EN:** This callable implements `APIServerReqTimeStats.get_first_token_latency` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.get_first_token_latency`，主要用于将数据转换为另一种表示。

### Lines 411-412: Method APIServerReqTimeStats.get_e2e_latency
```python
    def get_e2e_latency(self):
        return self.finished_time - self.created_time
```
**EN:** This callable implements `APIServerReqTimeStats.get_e2e_latency` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.get_e2e_latency`，主要用于获取某个值或派生视图。

### Lines 414-415: Method APIServerReqTimeStats.get_decode_latency
```python
    def get_decode_latency(self):
        return self.finished_time - self.first_token_time
```
**EN:** This callable implements `APIServerReqTimeStats.get_decode_latency` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.get_decode_latency`，主要用于获取某个值或派生视图。

### Lines 417-418: Method APIServerReqTimeStats.get_response_sent_to_client_realtime
```python
    def get_response_sent_to_client_realtime(self):
        return convert_time_to_realtime(self.response_sent_to_client_time)
```
**EN:** This callable implements `APIServerReqTimeStats.get_response_sent_to_client_realtime` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.get_response_sent_to_client_realtime`，主要用于将数据转换为另一种表示。

### Lines 420-444: Method APIServerReqTimeStats.convert_to_output_meta_info
```python
    def convert_to_output_meta_info(
        self, scheduler_time_stats=None, completion_tokens=0
    ):
        meta_info = {}
        if self.created_time > 0.0:
            meta_info["request_received_ts"] = convert_time_to_realtime(
                self.created_time
            )
        if self.api_server_dispatch_finish_time > 0.0:
            meta_info["api_server_dispatch_finish_ts"] = convert_time_to_realtime(
                self.api_server_dispatch_finish_time
            )
        if self.response_sent_to_client_time > 0.0:
            meta_info["response_sent_to_client_ts"] = convert_time_to_realtime(
                self.response_sent_to_client_time
            )
        if self.finished_time > 0.0:
            meta_info["request_finished_ts"] = convert_time_to_realtime(
                self.finished_time
            )

        decode_latency = self.get_decode_latency()
        if decode_latency > 0.0 and completion_tokens > 1:
            meta_info["decode_throughput"] = (completion_tokens - 1) / decode_latency
        return meta_info
```
**EN:** This callable implements `APIServerReqTimeStats.convert_to_output_meta_info`. It takes `scheduler_time_stats`, `completion_tokens` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.convert_to_output_meta_info`。它接收 `scheduler_time_stats`, `completion_tokens`，主要用于将数据转换为另一种表示。

### Lines 446-473: Method APIServerReqTimeStats.convert_to_gen_ai_span_attrs
```python
    def convert_to_gen_ai_span_attrs(self):
        span_attrs = {}
        if self.first_token_time and self.created_time:
            span_attrs[SpanAttributes.GEN_AI_LATENCY_TIME_TO_FIRST_TOKEN] = (
                self.first_token_time - self.created_time
            )

        if self.finished_time and self.created_time:
            span_attrs[SpanAttributes.GEN_AI_LATENCY_E2E] = (
                self.finished_time - self.created_time
            )

        if self.first_token_time and self.finished_time:
            span_attrs[SpanAttributes.GEN_AI_LATENCY_TIME_IN_MODEL_DECODE] = (
                self.finished_time - self.first_token_time
            )

        if self.api_server_dispatch_finish_time and self.finished_time:
            span_attrs[SpanAttributes.GEN_AI_LATENCY_TIME_IN_MODEL_INFERENCE] = (
                self.finished_time - self.api_server_dispatch_finish_time
            )

        if self.api_server_dispatch_finish_time and self.first_token_time:
            span_attrs[SpanAttributes.GEN_AI_LATENCY_TIME_IN_MODEL_PREFILL] = (
                self.first_token_time - self.api_server_dispatch_finish_time
            )

        return span_attrs
```
**EN:** This callable implements `APIServerReqTimeStats.convert_to_gen_ai_span_attrs` and mainly converts data into another representation. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `APIServerReqTimeStats.convert_to_gen_ai_span_attrs`，主要用于将数据转换为另一种表示。 在这一范围内，它会记录指标或追踪信号。

### Lines 476-485: Class DPControllerReqTimeStats
```python
@dataclass
class DPControllerReqTimeStats(ReqTimeStatsBase):
    # propagated from tokenizer/grpc_server, get by time.perf_counter()
    created_time: float = 0.0
    api_server_dispatch_time: float = 0.0

    # new timestamp, get by time.perf_counter()
    dpc_dispatch_time: float = 0.0
    dpc_dispatch_finish_time: float = 0.0

```
**EN:** This range introduces `DPControllerReqTimeStats` and defines the structure or metadata that its methods rely on. In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `DPControllerReqTimeStats`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 486-496: Method DPControllerReqTimeStats.__getstate__
```python
    def __getstate__(self) -> object:
        state = {}
        # send to Scheduler
        # If necessary, can propagate the timestamp here, for example:
        # state = {
        #     "created_time": self.created_time,
        #     "api_server_dispatch_time": self.api_server_dispatch_time,
        #     "dpc_dispatch_time": self.dpc_dispatch_time,
        # }
        state.update(super().__getstate__())
        return state
```
**EN:** This callable implements `DPControllerReqTimeStats.__getstate__` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `DPControllerReqTimeStats.__getstate__`，主要用于获取某个值或派生视图。

### Lines 498-507: Method DPControllerReqTimeStats.set_dp_dispatch_time
```python
    def set_dp_dispatch_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.dpc_dispatch_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_slice_start(
                RequestStage.DPC_DISPATCH.stage_name,
                RequestStage.DPC_DISPATCH.level,
                convert_time_to_realtime_ns(ts),
            )
```
**EN:** This callable implements `DPControllerReqTimeStats.set_dp_dispatch_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `DPControllerReqTimeStats.set_dp_dispatch_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 509-519: Method DPControllerReqTimeStats.set_dp_dispatch_finish_time
```python
    def set_dp_dispatch_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.dpc_dispatch_finish_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_slice_end(
                RequestStage.DPC_DISPATCH.stage_name,
                RequestStage.DPC_DISPATCH.level,
                convert_time_to_realtime_ns(ts),
                thread_finish_flag=True,
            )
```
**EN:** This callable implements `DPControllerReqTimeStats.set_dp_dispatch_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `DPControllerReqTimeStats.set_dp_dispatch_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 522-578: Class SchedulerReqTimeStats
```python
@dataclass
class SchedulerReqTimeStats(ReqTimeStatsBase):
    """
    Store the timestamps for each stage of a request.

    Unified: wait_queue -> forward -> completion
    Prefill: bootstrap_queue -> wait_queue -> forward -> transfer_queue -> completion
    Decode: prealloc_queue -> transfer_queue -> wait_queue -> forward -> completion
    """

    # Placeholder: not used currently
    # propagated from tokenizer/grpc_server or dp controller
    created_time: float = 0.0
    api_server_dispatch_time: float = 0.0
    dpc_dispatch_time: float = 0.0

    # common, get by time.perf_counter()
    wait_queue_entry_time: float = 0.0
    forward_entry_time: float = 0.0
    prefill_finished_time: float = 0.0
    completion_time: float = 0.0

    # prefill node, get by time.perf_counter()
    prefill_bootstrap_queue_entry_time: float = 0.0
    prefill_transfer_queue_entry_time: float = 0.0
    prefill_kv_transfer_finish_time: float = 0.0

    # decode node, get by time.perf_counter()
    decode_prealloc_queue_entry_time: float = 0.0
    decode_transfer_queue_entry_time: float = 0.0
    decode_prebuilt_finish_time: float = 0.0

    # bootstrap sub-phase tracking (PD disagg)
    bootstrap_done_time: float = 0.0

    # only for request tracing
    scheduler_recv_time: float = 0.0
    last_chunked_prefill_finish_time: float = 0.0
    last_decode_finish_time: float = 0.0
    decode_ct: int = 0
    last_decode_scheduled_time: float = 0.0
    last_forward_entry_time: float = 0.0
    last_prefill_finished_time: float = 0.0
    run_batch_cpu_start_time: float = 0.0

    # speculative decoding
    spec_draft_start_time: float = 0.0
    spec_verify_start_time: float = 0.0
    spec_draft_extend_start_time: float = 0.0

    # other
    transfer_speed_gb_s: float = 0.0
    transfer_total_mb: float = 0.0

    # Number of prefill retries for this request
    prefill_retry_count: int = 0

```
**EN:** This range introduces `SchedulerReqTimeStats` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Store the timestamps for each stage of a request." In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `SchedulerReqTimeStats`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 579-590: Method SchedulerReqTimeStats.__getstate__
```python
    def __getstate__(self) -> object:
        # send to detokenizer/tokenizer
        if not self.enable_metrics:
            return {}

        state = {
            "wait_queue_entry_time": self.wait_queue_entry_time,
            "forward_entry_time": self.forward_entry_time,
            "prefill_finished_time": self.prefill_finished_time,
            "diff_realtime_monotonic": global_diff_realtime_monotonic,
        }
        return state
```
**EN:** This callable implements `SchedulerReqTimeStats.__getstate__` and mainly retrieves a value or derived view. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.__getstate__`，主要用于获取某个值或派生视图。 在这一范围内，它会记录指标或追踪信号。

### Lines 592-595: Method SchedulerReqTimeStats.set_scheduler_recv_time
```python
    def set_scheduler_recv_time(self, ts=None):
        calibrate_time_diff()
        ts = ts or time.perf_counter()
        self.scheduler_recv_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_scheduler_recv_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_scheduler_recv_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 597-599: Method SchedulerReqTimeStats.set_spec_draft_start_time
```python
    def set_spec_draft_start_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.spec_draft_start_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_draft_start_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_draft_start_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 601-606: Method SchedulerReqTimeStats.set_spec_draft_end_time
```python
    def set_spec_draft_end_time(self, ts=None):
        ts = ts or time.perf_counter()

        if self.trace_ctx.tracing_enable:
            stage = RequestStage.SPEC_DRAFT
            self.trace_slice(stage, self.spec_draft_start_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_draft_end_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_draft_end_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 608-610: Method SchedulerReqTimeStats.set_spec_verify_start_time
```python
    def set_spec_verify_start_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.spec_verify_start_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_verify_start_time`. It takes `ts` and mainly validates inputs and invariants.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_verify_start_time`。它接收 `ts`，主要用于校验输入与不变量。

### Lines 612-634: Method SchedulerReqTimeStats.set_spec_verify_end_time
```python
    def set_spec_verify_end_time(
        self,
        ts=None,
        num_correct_drafts: int = 0,
        # FIXME: backward-compat alias, remove in next release.
        accepted_tokens: Optional[int] = None,
    ):
        if accepted_tokens is not None:
            num_correct_drafts = accepted_tokens
        ts = ts or time.perf_counter()

        if self.trace_ctx.tracing_enable:
            stage = RequestStage.SPEC_VERIFY
            self.trace_slice(
                stage,
                self.spec_verify_start_time,
                ts,
                {
                    "num_correct_drafts": num_correct_drafts,
                    # FIXME: backward-compat alias, remove in next release.
                    "accepted_tokens": num_correct_drafts,
                },
            )
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_verify_end_time`. It takes `ts`, `num_correct_drafts`, `accepted_tokens` and mainly validates inputs and invariants. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_verify_end_time`。它接收 `ts`, `num_correct_drafts`, `accepted_tokens`，主要用于校验输入与不变量。 在这一范围内，它会记录指标或追踪信号。

### Lines 636-638: Method SchedulerReqTimeStats.set_spec_draft_extend_start_time
```python
    def set_spec_draft_extend_start_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.spec_draft_extend_start_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_draft_extend_start_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_draft_extend_start_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 640-645: Method SchedulerReqTimeStats.set_spec_draft_extend_end_time
```python
    def set_spec_draft_extend_end_time(self, ts=None):
        ts = ts or time.perf_counter()

        if self.trace_ctx.tracing_enable:
            stage = RequestStage.SPEC_DRAFT_EXTEND
            self.trace_slice(stage, self.spec_draft_extend_start_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_spec_draft_extend_end_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_spec_draft_extend_end_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 647-649: Method SchedulerReqTimeStats.set_run_batch_cpu_start_time
```python
    def set_run_batch_cpu_start_time(self, ts=None, attrs=None):
        ts = ts or time.perf_counter()
        self.run_batch_cpu_start_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_run_batch_cpu_start_time`. It takes `ts`, `attrs` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_run_batch_cpu_start_time`。它接收 `ts`, `attrs`，主要用于将配置写入可变状态。

### Lines 651-657: Method SchedulerReqTimeStats.set_run_batch_cpu_end_time
```python
    def set_run_batch_cpu_end_time(self, ts=None, attrs=None):
        ts = ts or time.perf_counter()
        if self.run_batch_cpu_start_time > 0.0:
            self.trace_slice(
                RequestStage.RUN_BATCH_CPU, self.run_batch_cpu_start_time, ts, attrs
            )
            self.run_batch_cpu_start_time = 0.0
```
**EN:** This callable implements `SchedulerReqTimeStats.set_run_batch_cpu_end_time`. It takes `ts`, `attrs` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_run_batch_cpu_end_time`。它接收 `ts`, `attrs`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 659-669: Method SchedulerReqTimeStats.set_retract_time
```python
    def set_retract_time(self, ts=None):
        ts = ts or time.perf_counter()
        # retract
        self.last_forward_entry_time = 0.0
        self.last_prefill_finished_time = 0.0
        self.last_chunked_prefill_finish_time = 0.0
        self.last_decode_finish_time = 0.0
        self.last_decode_scheduled_time = 0.0

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.trace_event("retract", 1, convert_time_to_realtime_ns(ts))
```
**EN:** This callable implements `SchedulerReqTimeStats.set_retract_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_retract_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 671-690: Method SchedulerReqTimeStats.set_wait_queue_entry_time
```python
    def set_wait_queue_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        if self.wait_queue_entry_time == 0.0:
            if self.enable_metrics or self.trace_ctx.tracing_enable:
                if self.disagg_mode == DisaggregationMode.PREFILL:
                    stage = RequestStage.PREFILL_BOOTSTRAP
                    slice_start_time = self.prefill_bootstrap_queue_entry_time
                elif self.disagg_mode == DisaggregationMode.DECODE:
                    stage = RequestStage.DECODE_TRANSFERRED
                    slice_start_time = self.decode_transfer_queue_entry_time
                else:
                    stage = RequestStage.REQUEST_PROCESS
                    slice_start_time = self.scheduler_recv_time

                self.observe_per_stage_req_latency(stage, ts - slice_start_time)
                self.trace_slice(stage, slice_start_time, ts)
        else:
            self.set_retract_time(ts)

        self.wait_queue_entry_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_wait_queue_entry_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_wait_queue_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 692-724: Method SchedulerReqTimeStats.set_forward_entry_time
```python
    def set_forward_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        if self.forward_entry_time == 0.0:
            self.forward_entry_time = ts
            self.last_forward_entry_time = ts

            if self.enable_metrics:
                self.metrics_collector.observe_queue_time(self.get_queueing_time())

            if self.enable_metrics or self.trace_ctx.tracing_enable:
                if self.disagg_mode == DisaggregationMode.DECODE:
                    stage = RequestStage.DECODE_WAITING
                else:
                    stage = RequestStage.PREFILL_WAITING
                slice_start_time = self.wait_queue_entry_time

                self.observe_per_stage_req_latency(stage, ts - slice_start_time)
                self.trace_slice(stage, slice_start_time, ts)

                if self.disagg_mode == DisaggregationMode.DECODE:
                    self.trace_ctx.trace_slice_start(
                        RequestStage.DECODE_FORWARD.stage_name,
                        RequestStage.DECODE_FORWARD.level,
                        convert_time_to_realtime_ns(ts),
                    )
                else:
                    self.trace_ctx.trace_slice_start(
                        RequestStage.PREFILL_FORWARD.stage_name,
                        RequestStage.PREFILL_FORWARD.level,
                        convert_time_to_realtime_ns(ts),
                    )
        elif self.last_forward_entry_time == 0.0:
            self.last_forward_entry_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_forward_entry_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_forward_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 726-736: Method SchedulerReqTimeStats.set_last_chunked_prefill_finish_time
```python
    def set_last_chunked_prefill_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        last_time = self.last_chunked_prefill_finish_time
        self.last_chunked_prefill_finish_time = ts

        if last_time == 0.0:
            last_time = self.last_forward_entry_time

        stage = RequestStage.PREFILL_CHUNKED_FORWARD
        self.observe_per_stage_req_latency(stage, ts - last_time)
        self.trace_slice(stage, last_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_last_chunked_prefill_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_last_chunked_prefill_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 738-779: Method SchedulerReqTimeStats.set_prefill_finished_time
```python
    def set_prefill_finished_time(self, ts=None):
        ts = ts or time.perf_counter()
        if self.prefill_finished_time == 0.0:
            self.prefill_finished_time = ts
            self.last_prefill_finished_time = ts

            stage = RequestStage.PREFILL_FORWARD
            self.observe_per_stage_req_latency(stage, ts - self.last_forward_entry_time)

            if self.trace_ctx.tracing_enable:
                if self.last_chunked_prefill_finish_time > 0:
                    self.trace_slice(
                        RequestStage.PREFILL_CHUNKED_FORWARD,
                        self.last_chunked_prefill_finish_time,
                        ts,
                    )

                self.trace_ctx.trace_slice_end(
                    stage.stage_name, stage.level, convert_time_to_realtime_ns(ts)
                )
                if (
                    self.disagg_mode == DisaggregationMode.NULL
                    and self.last_decode_scheduled_time > 0
                ):
                    self.trace_ctx.trace_slice_start(
                        RequestStage.DECODE_FORWARD.stage_name,
                        RequestStage.DECODE_FORWARD.level,
                        convert_time_to_realtime_ns(ts),
                    )
        elif self.last_prefill_finished_time == 0.0:
            # retract
            self.last_prefill_finished_time = ts
            if self.last_chunked_prefill_finish_time > 0:
                self.trace_slice(
                    RequestStage.PREFILL_CHUNKED_FORWARD,
                    self.last_chunked_prefill_finish_time,
                    ts,
                )
            else:
                self.trace_slice(
                    RequestStage.PREFILL_FORWARD, self.last_forward_entry_time, ts
                )
```
**EN:** This callable implements `SchedulerReqTimeStats.set_prefill_finished_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_prefill_finished_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 781-802: Method SchedulerReqTimeStats.set_last_decode_finish_time
```python
    def set_last_decode_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        last_time = self.last_decode_finish_time
        self.last_decode_finish_time = ts

        if self.enable_metrics or self.trace_ctx.tracing_enable:
            if last_time == 0.0:
                if self.disagg_mode == DisaggregationMode.DECODE:
                    last_time = self.decode_prebuilt_finish_time
                else:
                    if (
                        self.last_decode_scheduled_time
                        < self.last_prefill_finished_time
                    ):
                        last_time = self.last_prefill_finished_time
                    else:
                        last_time = self.last_decode_scheduled_time
            stage = RequestStage.DECODE_LOOP
            self.observe_per_stage_req_latency(stage, ts - last_time)
            attrs = {"decode_ct": self.decode_ct}
            self.trace_slice(stage, last_time, ts, attrs)
            self.decode_ct += 1
```
**EN:** This callable implements `SchedulerReqTimeStats.set_last_decode_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_last_decode_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 804-829: Method SchedulerReqTimeStats.set_last_scheduled_time
```python
    def set_last_scheduled_time(self, forward_mode: ForwardMode, ts=None, attrs=None):
        ts = ts or time.perf_counter()

        if self.trace_ctx.tracing_enable:
            if (
                self.disagg_mode == DisaggregationMode.NULL
                and forward_mode.is_decode()
                and self.last_decode_scheduled_time == 0.0
                and self.last_prefill_finished_time > 0
            ):
                self.trace_slice(
                    RequestStage.DECODE_WAITING, self.last_prefill_finished_time, ts
                )
                self.trace_ctx.trace_slice_start(
                    RequestStage.DECODE_FORWARD.stage_name,
                    RequestStage.DECODE_FORWARD.level,
                    convert_time_to_realtime_ns(ts),
                )
                self.last_decode_finish_time = ts

            self.trace_ctx.trace_event(
                "schedule", 3, convert_time_to_realtime_ns(ts), attrs
            )

        if forward_mode.is_decode():
            self.last_decode_scheduled_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_last_scheduled_time`. It takes `forward_mode`, `ts`, `attrs` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_last_scheduled_time`。它接收 `forward_mode`, `ts`, `attrs`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 831-836: Method SchedulerReqTimeStats.set_completion_time
```python
    def set_completion_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.completion_time = ts

        if self.trace_ctx.tracing_enable:
            self.trace_ctx.abort()
```
**EN:** This callable implements `SchedulerReqTimeStats.set_completion_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_completion_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 838-897: Method SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics (part 1/2)
```python
    def compute_and_observe_kv_transfer_metrics(
        self,
        transfer_metric: KVTransferMetric,
    ) -> Optional[dict]:
        """Compute KV transfer metrics and observe them via the metrics collector.

        Returns a dict with latency_ms, total_mb, speed_gb_s if computable, else None.
        """
        result = {}
        if transfer_metric.transfer_total_bytes is None:
            return result if result else None

        # Transfer latency, size, and speed
        if transfer_metric.transfer_latency_s is not None:
            transfer_latency_s = transfer_metric.transfer_latency_s
        else:
            if self.prefill_transfer_queue_entry_time <= 0 or self.completion_time <= 0:
                return result if result else None
            # Note: This only capture the last chunk time
            transfer_latency_s = (
                self.completion_time - self.prefill_transfer_queue_entry_time
            )

        if transfer_latency_s > 0:
            latency_ms = transfer_latency_s * 1000

            total_bytes = transfer_metric.transfer_total_bytes
            total_mb = total_bytes / (1024 * 1024)
            self.transfer_total_mb = total_mb

            speed_gb_s = 0.0
            if transfer_latency_s > 0:
                speed_gb_s = (total_mb / 1024) / transfer_latency_s
                self.transfer_speed_gb_s = speed_gb_s

            result["latency_ms"] = latency_ms
            result["total_mb"] = total_mb
            result["speed_gb_s"] = speed_gb_s

            if self.enable_metrics:
                self.metrics_collector.observe_kv_transfer_metrics(
                    latency_ms=latency_ms,
                    total_mb=total_mb,
                    speed_gb_s=speed_gb_s,
                )

        # Bootstrap and alloc durations
        if (
            self.prefill_bootstrap_queue_entry_time > 0
            and self.bootstrap_done_time > 0
            and self.wait_queue_entry_time > 0
        ):
            bootstrap_ms = (
                self.bootstrap_done_time - self.prefill_bootstrap_queue_entry_time
            ) * 1000
            alloc_ms = (self.wait_queue_entry_time - self.bootstrap_done_time) * 1000

            result["bootstrap_ms"] = bootstrap_ms
            result["alloc_ms"] = alloc_ms

```
**EN:** This callable implements `SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics`. It takes `transfer_metric` and mainly implements compute and observe kv transfer metrics. The docstring states: "Compute KV transfer metrics and observe them via the metrics collector." This chunk is part 1 of 2 for the same logical block. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics`。它接收 `transfer_metric`，主要用于实现 compute and observe kv transfer metrics 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 898-904: Method SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics (part 2/2)
```python
            if self.enable_metrics:
                self.metrics_collector.observe_kv_transfer_bootstrap(
                    bootstrap_ms=bootstrap_ms,
                    alloc_ms=alloc_ms,
                )

        return result if result else None
```
**EN:** This callable implements `SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics`. It takes `transfer_metric` and mainly implements compute and observe kv transfer metrics. The docstring states: "Compute KV transfer metrics and observe them via the metrics collector." This chunk is part 2 of 2 for the same logical block. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.compute_and_observe_kv_transfer_metrics`。它接收 `transfer_metric`，主要用于实现 compute and observe kv transfer metrics 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会记录指标或追踪信号。

### Lines 906-909: Method SchedulerReqTimeStats.set_quick_finish_time
```python
    def set_quick_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.set_completion_time(ts)
        self.forward_entry_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_quick_finish_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_quick_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 911-917: Method SchedulerReqTimeStats.set_prefill_bootstrap_queue_entry_time
```python
    def set_prefill_bootstrap_queue_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.prefill_bootstrap_queue_entry_time = ts

        stage = RequestStage.PREFILL_PREPARE
        self.observe_per_stage_req_latency(stage, ts - self.scheduler_recv_time)
        self.trace_slice(stage, self.scheduler_recv_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_prefill_bootstrap_queue_entry_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_prefill_bootstrap_queue_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 919-921: Method SchedulerReqTimeStats.set_prefill_transfer_queue_entry_time
```python
    def set_prefill_transfer_queue_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.prefill_transfer_queue_entry_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_prefill_transfer_queue_entry_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_prefill_transfer_queue_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 923-931: Method SchedulerReqTimeStats.set_prefill_kv_transfer_finish_time
```python
    def set_prefill_kv_transfer_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.prefill_kv_transfer_finish_time = ts

        stage = RequestStage.PREFILL_TRANSFER_KV_CACHE
        self.observe_per_stage_req_latency(
            stage, ts - self.prefill_transfer_queue_entry_time
        )
        self.trace_slice(stage, self.prefill_transfer_queue_entry_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_prefill_kv_transfer_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_prefill_kv_transfer_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 933-939: Method SchedulerReqTimeStats.set_decode_prealloc_queue_entry_time
```python
    def set_decode_prealloc_queue_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.decode_prealloc_queue_entry_time = ts

        stage = RequestStage.DECODE_PREPARE
        self.observe_per_stage_req_latency(stage, ts - self.scheduler_recv_time)
        self.trace_slice(stage, self.scheduler_recv_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_decode_prealloc_queue_entry_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_decode_prealloc_queue_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 941-959: Method SchedulerReqTimeStats.set_decode_transfer_queue_entry_time
```python
    def set_decode_transfer_queue_entry_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.decode_transfer_queue_entry_time = ts

        stage = RequestStage.DECODE_BOOTSTRAP
        self.observe_per_stage_req_latency(
            stage, ts - self.decode_prealloc_queue_entry_time
        )
        self.trace_slice(stage, self.decode_prealloc_queue_entry_time, ts)

        if self.enable_metrics and self.bootstrap_done_time > 0:
            bootstrap_ms = (
                self.bootstrap_done_time - self.decode_prealloc_queue_entry_time
            ) * 1000
            alloc_ms = (ts - self.bootstrap_done_time) * 1000
            self.metrics_collector.observe_kv_transfer_bootstrap(
                bootstrap_ms=bootstrap_ms,
                alloc_ms=alloc_ms,
            )
```
**EN:** This callable implements `SchedulerReqTimeStats.set_decode_transfer_queue_entry_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_decode_transfer_queue_entry_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 961-964: Method SchedulerReqTimeStats.set_bootstrap_done_time
```python
    def set_bootstrap_done_time(self, ts=None):
        ts = ts or time.perf_counter()
        if self.bootstrap_done_time == 0.0:
            self.bootstrap_done_time = ts
```
**EN:** This callable implements `SchedulerReqTimeStats.set_bootstrap_done_time`. It takes `ts` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_bootstrap_done_time`。它接收 `ts`，主要用于将配置写入可变状态。

### Lines 966-972: Method SchedulerReqTimeStats.set_decode_prebuilt_finish_time
```python
    def set_decode_prebuilt_finish_time(self, ts=None):
        ts = ts or time.perf_counter()
        self.decode_prebuilt_finish_time = ts

        stage = RequestStage.DECODE_FAKE_OUTPUT
        self.observe_per_stage_req_latency(stage, ts - self.last_forward_entry_time)
        self.trace_slice(stage, self.last_forward_entry_time, ts)
```
**EN:** This callable implements `SchedulerReqTimeStats.set_decode_prebuilt_finish_time`. It takes `ts` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.set_decode_prebuilt_finish_time`。它接收 `ts`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 974-975: Method SchedulerReqTimeStats.get_queueing_time
```python
    def get_queueing_time(self) -> float:
        return self.forward_entry_time - self.wait_queue_entry_time
```
**EN:** This callable implements `SchedulerReqTimeStats.get_queueing_time` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.get_queueing_time`，主要用于获取某个值或派生视图。

### Lines 977-1036: Method SchedulerReqTimeStats.convert_to_duration (part 1/2)
```python
    def convert_to_duration(self) -> str:
        if self.disagg_mode == DisaggregationMode.NULL:
            queue_duration = self.duration_between(
                self.wait_queue_entry_time, self.forward_entry_time
            )
            forward_duration = self.duration_between(
                self.forward_entry_time, self.completion_time
            )

            if SGLANG_TEST_REQUEST_TIME_STATS:
                assert (
                    queue_duration >= 0 and forward_duration >= 0
                ), f"queue_duration={queue_duration} < 0 or forward_duration={forward_duration} < 0"

            return f"queue_duration={self.format_duration(queue_duration)}, forward_duration={self.format_duration(forward_duration)}, entry_time={self.format_wallclock(self.wait_queue_entry_time)}"
        elif self.disagg_mode == DisaggregationMode.PREFILL:
            bootstrap_queue_duration = self.duration_between(
                self.prefill_bootstrap_queue_entry_time, self.wait_queue_entry_time
            )
            queue_duration = self.duration_between(
                self.wait_queue_entry_time, self.forward_entry_time
            )
            forward_duration = self.duration_between(
                self.forward_entry_time, self.completion_time
            )

            if SGLANG_TEST_REQUEST_TIME_STATS:
                if self.wait_queue_entry_time > 0:
                    assert (
                        bootstrap_queue_duration >= 0
                        and queue_duration >= 0
                        and forward_duration >= 0
                    ), f"bootstrap_queue_duration={bootstrap_queue_duration} < 0 or queue_duration={queue_duration} < 0 or forward_duration={forward_duration} < 0"

            # Break down bootstrap_queue_duration into sub-phases
            if self.bootstrap_done_time > 0:
                bootstrap_duration = self.duration_between(
                    self.prefill_bootstrap_queue_entry_time, self.bootstrap_done_time
                )
                alloc_wait_duration = self.duration_between(
                    self.bootstrap_done_time, self.wait_queue_entry_time
                )
                if SGLANG_TEST_REQUEST_TIME_STATS:
                    assert (
                        bootstrap_duration >= 0 and alloc_wait_duration >= 0
                    ), f"bootstrap_duration={bootstrap_duration} < 0 or alloc_wait_duration={alloc_wait_duration} < 0"
                bootstrap_fields = (
                    f"bootstrap_duration={self.format_duration(bootstrap_duration)}, "
                    f"alloc_wait_duration={self.format_duration(alloc_wait_duration)}, "
                )
            else:
                bootstrap_fields = f"bootstrap_queue_duration={self.format_duration(bootstrap_queue_duration)}, "

            return (
                f"{bootstrap_fields}"
                f"queue_duration={self.format_duration(queue_duration)}, "
                f"forward_duration={self.format_duration(forward_duration)}, "
                f"entry_time={self.format_wallclock(self.prefill_bootstrap_queue_entry_time)}, "
                f"transfer_speed={self.transfer_speed_gb_s:.2f} GB/s, "
                f"transfer_total={self.transfer_total_mb:.2f} MB, "
```
**EN:** This callable implements `SchedulerReqTimeStats.convert_to_duration` and mainly converts data into another representation. This chunk is part 1 of 2 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.convert_to_duration`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1037-1093: Method SchedulerReqTimeStats.convert_to_duration (part 2/2)
```python
                f"#retries={self.prefill_retry_count}"
            )
        elif self.disagg_mode == DisaggregationMode.DECODE:
            prealloc_duration = self.duration_between(
                self.decode_prealloc_queue_entry_time,
                self.decode_transfer_queue_entry_time,
            )
            transfer_duration = self.duration_between(
                self.decode_transfer_queue_entry_time,
                self.wait_queue_entry_time,
            )
            queue_duration = self.duration_between(
                self.wait_queue_entry_time,
                self.forward_entry_time,
            )
            forward_duration = self.duration_between(
                self.forward_entry_time,
                self.completion_time,
            )

            if SGLANG_TEST_REQUEST_TIME_STATS:
                if self.wait_queue_entry_time > 0:
                    assert (
                        prealloc_duration >= 0
                        and transfer_duration >= 0
                        and queue_duration >= 0
                        and forward_duration >= 0
                    ), f"prealloc_duration={prealloc_duration} < 0 or transfer_duration={transfer_duration} < 0 or queue_duration={queue_duration} < 0 or forward_duration={forward_duration} < 0. {self=}"

            # Break down prealloc_duration into sub-phases
            if self.bootstrap_done_time > 0:
                bootstrap_duration = self.duration_between(
                    self.decode_prealloc_queue_entry_time, self.bootstrap_done_time
                )
                alloc_wait_duration = self.duration_between(
                    self.bootstrap_done_time, self.decode_transfer_queue_entry_time
                )
                if SGLANG_TEST_REQUEST_TIME_STATS:
                    assert (
                        bootstrap_duration >= 0 and alloc_wait_duration >= 0
                    ), f"bootstrap_duration={bootstrap_duration} < 0 or alloc_wait_duration={alloc_wait_duration} < 0"
                prealloc_fields = (
                    f"bootstrap_duration={self.format_duration(bootstrap_duration)}, "
                    f"alloc_wait_duration={self.format_duration(alloc_wait_duration)}, "
                )
            else:
                prealloc_fields = f"prealloc_queue_duration={self.format_duration(prealloc_duration)}, "

            return (
                f"{prealloc_fields}"
                f"transfer_duration={self.format_duration(transfer_duration)}, "
                f"queue_duration={self.format_duration(queue_duration)}, "
                f"forward_duration={self.format_duration(forward_duration)}, "
                f"entry_time={self.format_wallclock(self.decode_prealloc_queue_entry_time)}"
            )
        else:
            return "Unknown Time Stats"
```
**EN:** This callable implements `SchedulerReqTimeStats.convert_to_duration` and mainly converts data into another representation. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.convert_to_duration`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1095-1110: Method SchedulerReqTimeStats.convert_to_output_meta_info
```python
    def convert_to_output_meta_info(self):
        meta_data = {}
        if self.forward_entry_time > 0.0:
            meta_data["forward_entry_time"] = convert_time_to_realtime(
                self.forward_entry_time
            )
        if self.prefill_finished_time > 0.0:
            meta_data["prefill_finished_time"] = convert_time_to_realtime(
                self.prefill_finished_time
            )
        meta_data.update(
            {
                "queue_time": self.get_queueing_time(),
            }
        )
        return meta_data
```
**EN:** This callable implements `SchedulerReqTimeStats.convert_to_output_meta_info` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.convert_to_output_meta_info`，主要用于将数据转换为另一种表示。

### Lines 1112-1113: Method SchedulerReqTimeStats.format_duration
```python
    def format_duration(self, duration: float) -> str:
        return f"{duration * 1e3:.2f}ms"
```
**EN:** This callable implements `SchedulerReqTimeStats.format_duration`. It takes `duration` and mainly implements format duration.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.format_duration`。它接收 `duration`，主要用于实现 format duration 相关逻辑。

### Lines 1115-1118: Method SchedulerReqTimeStats.duration_between
```python
    def duration_between(self, start: float, end: float) -> float:
        if start <= 0 or end <= 0:
            return 0.0
        return end - start
```
**EN:** This callable implements `SchedulerReqTimeStats.duration_between`. It takes `start`, `end` and mainly implements duration between.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.duration_between`。它接收 `start`, `end`，主要用于实现 duration between 相关逻辑。

### Lines 1120-1122: Method SchedulerReqTimeStats.format_wallclock
```python
    @staticmethod
    def format_wallclock(perf_counter_time: float) -> str:
        return f"{convert_time_to_realtime(perf_counter_time):.3f}"
```
**EN:** This callable implements `SchedulerReqTimeStats.format_wallclock`. It takes `perf_counter_time` and mainly implements format wallclock.
**CN:** 这一可调用对象实现了 `SchedulerReqTimeStats.format_wallclock`。它接收 `perf_counter_time`，主要用于实现 format wallclock 相关逻辑。

### Lines 1125-1141: Function set_schedule_time_batch
```python
def set_schedule_time_batch(batch: ScheduleBatch):
    # only for tracing
    if not get_global_tracing_enabled():
        return

    ts = time.perf_counter()
    bid = uuid.uuid4().hex[:8]
    _attrs = {"bid": bid, "batch_size": len(batch.reqs)}
    if batch.forward_mode.is_decode():
        _attrs["forward_mode"] = "decode"
    elif batch.forward_mode.is_prefill():
        _attrs["forward_mode"] = "prefill"
    elif batch.forward_mode.is_prebuilt():
        _attrs["forward_mode"] = "prebuilt"

    for req in batch.reqs:
        req.time_stats.set_last_scheduled_time(batch.forward_mode, ts, _attrs)
```
**EN:** This callable implements `set_schedule_time_batch`. It takes `batch` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `set_schedule_time_batch`。它接收 `batch`，主要用于将配置写入可变状态。

### Lines 1144-1161: Function set_time_batch
```python
def set_time_batch(
    reqs: List[Any],
    set_func: str,
    trace_only: bool = False,
    attrs: Optional[Dict[str, Any]] = None,
):
    if reqs is None or len(reqs) == 0:
        return
    if trace_only and not get_global_tracing_enabled():
        return

    ts = time.perf_counter()
    for req in reqs:
        method = getattr(req.time_stats, set_func)
        if attrs is None:
            method(ts)
        else:
            method(ts, attrs)
```
**EN:** This callable implements `set_time_batch`. It takes `reqs`, `set_func`, `trace_only`, `attrs` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `set_time_batch`。它接收 `reqs`, `set_func`, `trace_only`, `attrs`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

## Key Concepts / 关键概念
- `SGLANG_TEST_REQUEST_TIME_STATS`: module constant or capability flag / 模块常量或能力标记
- `calibrate_time_diff`: implements calibrate time diff / 实现 calibrate time diff 相关逻辑
- `convert_time_to_realtime`: converts data into another representation / 将数据转换为另一种表示
- `convert_time_to_realtime_ns`: converts data into another representation / 将数据转换为另一种表示
- `convert_time_cross_thread`: implements convert time cross thread / 实现 convert time cross thread 相关逻辑
- `RequestStageConfig`: core class or state container / 核心类或状态容器
- `RequestStage`: core class or state container / 核心类或状态容器
- `ReqTimeStatsBase`: core class or state container / 核心类或状态容器
- `APIServerReqTimeStats`: core class or state container / 核心类或状态容器
- `DPControllerReqTimeStats`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `time`, `uuid`, `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.utils`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.observability.metrics_collector`, `sglang.srt.observability.trace`, `sglang.srt.utils`, `sglang.srt.disaggregation.base.conn`, `sglang.srt.managers.schedule_batch`
