# request_receiver.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/request_receiver.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements request receiver logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 请求 receiver 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-12: Provide supporting module logic / 提供辅助模块逻辑
```python
from dataclasses import dataclass
from http import HTTPStatus
from typing import (
    TYPE_CHECKING,
    Any,
    Callable,
    List,
    Optional,
    Union,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 14-15: Import runtime dependencies / 导入运行时依赖
```python
import zmq
from torch.distributed import barrier
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 17-31: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.disaggregation.utils import prepare_abort
from sglang.srt.managers.io_struct import (
    BatchTokenizedEmbeddingReqInput,
    BatchTokenizedGenerateReqInput,
    TokenizedEmbeddingReqInput,
    TokenizedGenerateReqInput,
)
from sglang.srt.managers.mm_utils import (
    has_shm_features,
    unwrap_shm_features,
)
from sglang.srt.utils import (
    broadcast_pyobj,
    point_to_point_pyobj,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 33-58: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.distributed.parallel_state_wrapper import ParallelState
    from sglang.srt.server_args import ServerArgs


@dataclass(kw_only=True, slots=True, frozen=True)
class SchedulerRequestReceiver:
    recv_from_tokenizer: zmq.Socket
    recv_from_rpc: Optional[zmq.Socket]
    recv_skipper: Any
    input_blocker: Any
    mm_receiver: Any
    ps: "ParallelState"
    tp_group: Any
    tp_cpu_group: Any
    attn_tp_group: Any
    attn_tp_cpu_group: Any
    attn_cp_group: Any
    attn_cp_cpu_group: Any
    world_group: Any
    server_args: "ServerArgs"
    model_config: "ModelConfig"
    max_recv_per_poll: int
    stream_output: Callable[..., None]
    get_last_forward_mode: Callable[[], Any]
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 60-63: Implement recv limit reached / 实现recv limit reached
```python
    def recv_limit_reached(self, num_recv_reqs: int) -> bool:
        if self.max_recv_per_poll < 0:
            return False
        return num_recv_reqs >= self.max_recv_per_poll
```
**EN:** This block implements the method `recv_limit_reached(num_recv_reqs)` on `SchedulerRequestReceiver`. It focuses on handling the request receiver responsibilities represented by `recv_limit_reached`, so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_limit_reached(num_recv_reqs)`。它围绕 `recv_limit_reached` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 65-100: Implement recv requests / 实现recv 请求
```python
    def recv_requests(
        self,
    ) -> List[Union[TokenizedGenerateReqInput, TokenizedEmbeddingReqInput, Any]]:
        """Receive results at tp_rank = 0 and broadcast it to all other TP ranks."""

        if self.recv_skipper is not None:
            if not self.recv_skipper.handle(self.get_last_forward_mode()):
                return []

        if self.ps.pp_rank == 0:
            if self.ps.attn_tp_rank == 0 and self.ps.attn_cp_rank == 0:
                recv_reqs = []

                while True:
                    try:
                        if self.recv_limit_reached(len(recv_reqs)):
                            break
                        recv_req = self.recv_from_tokenizer.recv_pyobj(zmq.NOBLOCK)
                    except zmq.ZMQError:
                        break
                    recv_reqs.append(recv_req)

                while True:
                    try:
                        if self.recv_limit_reached(len(recv_reqs)):
                            break
                        recv_rpc = self.recv_from_rpc.recv_pyobj(zmq.NOBLOCK)
                    except zmq.ZMQError:
                        break
                    recv_reqs.append(recv_rpc)
            else:
                recv_reqs = None
        else:
            if self.ps.attn_tp_rank == 0 and self.ps.attn_cp_rank == 0:
                dp_offset = self.ps.attn_dp_rank * self.ps.attn_tp_size
                recv_reqs = point_to_point_pyobj(
```
**EN:** This block implements the method `recv_requests()` on `SchedulerRequestReceiver`. It focuses on Receive results at tp_rank = 0 and broadcast it to all other TP ranks., so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_requests()`。它围绕 `recv_requests` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 101-136: Continue recv requests / 继续说明recv 请求
```python
                    [],
                    self.ps.pp_rank * self.ps.tp_size + dp_offset,
                    self.world_group.cpu_group,
                    (self.ps.pp_rank - 1) * self.ps.tp_size + dp_offset,
                    self.ps.pp_rank * self.ps.tp_size + dp_offset,
                )
            else:
                recv_reqs = None

        if self.input_blocker is not None:
            recv_reqs = self.input_blocker.handle(recv_reqs)

        if self.server_args.enable_dp_attention:
            if self.ps.attn_tp_rank == 0 and self.ps.attn_cp_rank == 0:
                work_reqs, control_reqs = self._split_work_and_control_reqs(recv_reqs)
            else:
                work_reqs = None
                control_reqs = None

            if self.ps.attn_tp_size != 1:
                work_reqs = broadcast_pyobj(
                    work_reqs,
                    self.attn_tp_group.rank,
                    self.attn_tp_cpu_group,
                    src=self.attn_tp_group.ranks[0],
                )

            if self.ps.attn_cp_size != 1:
                work_reqs = broadcast_pyobj(
                    work_reqs,
                    self.attn_cp_group.rank,
                    self.attn_cp_cpu_group,
                    src=self.attn_cp_group.ranks[0],
                )

            # When dp_attention_local_control_broadcast is enabled, each DP
```
**EN:** This block implements the method `recv_requests()` on `SchedulerRequestReceiver`. It focuses on Receive results at tp_rank = 0 and broadcast it to all other TP ranks., so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_requests()`。它围绕 `recv_requests` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 137-172: Continue recv requests / 继续说明recv 请求
```python
            # group leader already receives control messages from the DP
            # controller, so we broadcast within attn_tp_group + attn_cp_group
            # instead of the full tp_group.  This avoids an expensive
            # all-ranks gloo sync.
            _local_ctrl = self.server_args.enable_dp_attention_local_control_broadcast
            if _local_ctrl:
                if self.ps.attn_tp_size != 1:
                    control_reqs = broadcast_pyobj(
                        control_reqs,
                        self.attn_tp_group.rank,
                        self.attn_tp_cpu_group,
                        src=self.attn_tp_group.ranks[0],
                    )
                if self.ps.attn_cp_size != 1:
                    control_reqs = broadcast_pyobj(
                        control_reqs,
                        self.attn_cp_group.rank,
                        self.attn_cp_cpu_group,
                        src=self.attn_cp_group.ranks[0],
                    )
            elif self.ps.tp_size != 1:
                control_reqs = broadcast_pyobj(
                    control_reqs,
                    self.tp_group.rank,
                    self.tp_cpu_group,
                    src=self.tp_group.ranks[0],
                )
            recv_reqs = work_reqs + control_reqs
        elif self.ps.tp_size != 1:
            recv_reqs = broadcast_pyobj(
                recv_reqs,
                self.tp_group.rank,
                self.tp_cpu_group,
                src=self.tp_group.ranks[0],
            )
```
**EN:** This block implements the method `recv_requests()` on `SchedulerRequestReceiver`. It focuses on Receive results at tp_rank = 0 and broadcast it to all other TP ranks., so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_requests()`。它围绕 `recv_requests` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 173-208: Continue recv requests / 继续说明recv 请求
```python
        # Process MM requests under EPD-disaggregation mode
        if (
            self.ps.pp_rank == 0
            and self.server_args.language_only
            and self.server_args.encoder_transfer_backend == "zmq_to_scheduler"
        ):
            recv_reqs, abort_reqs = self.mm_receiver.process_waiting_requests(recv_reqs)
            for req, error_msg, error_code in abort_reqs:
                status_code = (
                    HTTPStatus.BAD_REQUEST
                    if error_code == 400
                    else HTTPStatus.INTERNAL_SERVER_ERROR
                )
                prepare_abort(req, error_msg, status_code=status_code)
                self.stream_output([req], req.return_logprob)

        # Unwrap shared memory features AFTER all broadcasts complete,
        # so that ShmPointerMMData metadata (not full tensor data) is what
        # gets serialized during broadcast_pyobj.
        if recv_reqs:
            # Barrier for the non-DP-attention path only: there is a single
            # broadcast_pyobj on tp_cpu_group where the source rank returns
            # the original objects immediately while other ranks are still in
            # pickle.loads (-> __setstate__ -> shm_open).  Without a barrier
            # the source can call materialize() / shm_unlink before others
            # open the segment.  recv_reqs is consistent across all ranks
            # here (same broadcast), so the guard is deadlock-free.
            #
            # Under DP-attention no barrier is needed: the control_reqs
            # broadcast on tp_cpu_group (step 3) is a collective that forces
            # every rank to complete the earlier attn_tp / attn_cp work_reqs
            # deserializations (steps 1-2, which call shm_open) before any
            # rank returns from step 3.  POSIX guarantees shm_unlink only
            # removes the name; already-open handles stay valid.
            if (
                not self.server_args.enable_dp_attention
```
**EN:** This block implements the method `recv_requests()` on `SchedulerRequestReceiver`. It focuses on Receive results at tp_rank = 0 and broadcast it to all other TP ranks., so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_requests()`。它围绕 `recv_requests` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 209-217: Continue recv requests / 继续说明recv 请求
```python
                and self.ps.tp_size > 1
                and self.model_config.is_multimodal
                and has_shm_features(recv_reqs)
            ):
                barrier(group=self.tp_cpu_group)
            for req in recv_reqs:
                unwrap_shm_features(req)

        return recv_reqs
```
**EN:** This block implements the method `recv_requests()` on `SchedulerRequestReceiver`. It focuses on Receive results at tp_rank = 0 and broadcast it to all other TP ranks., so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `recv_requests()`。它围绕 `recv_requests` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

### Lines 219-246: Implement split work and control reqs / 实现split work and control reqs
```python
    def _split_work_and_control_reqs(self, recv_reqs: List):
        work_reqs = [
            req
            for req in recv_reqs
            if isinstance(
                req,
                (
                    TokenizedGenerateReqInput,
                    TokenizedEmbeddingReqInput,
                    BatchTokenizedGenerateReqInput,
                    BatchTokenizedEmbeddingReqInput,
                ),
            )
        ]
        control_reqs = [
            req
            for req in recv_reqs
            if not isinstance(
                req,
                (
                    TokenizedGenerateReqInput,
                    TokenizedEmbeddingReqInput,
                    BatchTokenizedGenerateReqInput,
                    BatchTokenizedEmbeddingReqInput,
                ),
            )
        ]
        return work_reqs, control_reqs
```
**EN:** This block implements the method `_split_work_and_control_reqs(recv_reqs)` on `SchedulerRequestReceiver`. It focuses on handling the request receiver responsibilities represented by `_split_work_and_control_reqs`, so the class can advance the request receiver workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerRequestReceiver` 上的方法 `_split_work_and_control_reqs(recv_reqs)`。它围绕 `_split_work_and_control_reqs` 所承担的 请求 receiver 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerRequestReceiver
- **Domain focus / 领域焦点**: request receiver / 请求 receiver
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, typing
- **Third-party / 第三方库**: __future__, http, torch.distributed, zmq
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.disaggregation.utils, sglang.srt.distributed.parallel_state_wrapper, sglang.srt.managers.io_struct, sglang.srt.managers.mm_utils, sglang.srt.server_args, sglang.srt.utils
